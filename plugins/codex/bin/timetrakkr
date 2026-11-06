#!/usr/bin/env node
"use strict";

// packages/agent-cli/src/cli.ts
var import_node_child_process2 = require("node:child_process");

// packages/agent-cli/src/browser.ts
var OPENABLE = /^https?:\/\/[A-Za-z0-9._~:/?&=%-]+$/;
function browserCommand(platform, url) {
  if (!OPENABLE.test(url)) {
    return null;
  }
  if (platform === "darwin") {
    return { command: "open", args: [url] };
  }
  if (platform === "win32") {
    return { command: "rundll32", args: ["url.dll,FileProtocolHandler", url] };
  }
  return { command: "xdg-open", args: [url] };
}

// packages/agent-cli/src/constants.ts
var HOOK_BUDGET_MS = 1200;
var NETWORK_BUDGET_MS = 900;
var STDIN_BUDGET_MS = 300;
var COMMAND_NETWORK_MS = 5e3;
var GIT_TIMEOUT_MS = 400;
var CONFIG_TTL_MS = 5 * 6e4;
var STALE_SESSION_MS = 10 * 6e4;
var RETRY_WINDOW_MS = 7 * 24 * 60 * 6e4;
var PRUNE_AFTER_MS = 30 * 24 * 60 * 6e4;
var TRANSCRIPT_READ_BYTES = 8 * 1024 * 1024;
var RECENT_MESSAGES = 500;
var REPLY_SLACK_MS = 5e3;
var RENEW_MARGIN_MS = 5 * 6e4;
var RENEW_POLL_MS = 40;
var EARLY_RENEW_WAIT_MS = 400;
var RENEW_TIMEOUT_MS = 1e4;
var LOCK_POLL_MS = 50;
var LOCK_STALE_MS = 15e3;
var LOGIN_TIMEOUT_MS = 5 * 6e4;
var LOGIN_LOCK_STALE_MS = LOGIN_TIMEOUT_MS + 6e4;
var LOGIN_LINK_WAIT_MS = 5e3;
var MAX_TOKEN_MODELS = 20;
var HOURS_PER_DAY = 8;
var DAYS_PER_WEEK = 5;
var MAX_ESTIMATE_SECONDS = 31536e3;
var MAX_ESTIMATE_LABEL = 200;
var ERROR_LOG_MAX_BYTES = 256 * 1024;
var MAPPING_FILE = ".timetrakkr.json";
var MAPPING_MAX_BYTES = 16 * 1024;
var MAX_REPOSITORY_LENGTH = 500;
var MAX_BRANCH_LENGTH = 255;
var MAX_MODEL_LENGTH = 100;
var SECONDS_PER_DAY = 86400;
var MAX_TURN_COUNT = 1e4;
var WINDOW_SLACK_SECONDS = 60;
var MAX_DAY_AGE_DAYS = 10;
var HOSTS = ["claude-code", "codex", "cursor"];
var HOST_LABELS = {
  "claude-code": "Claude Code",
  codex: "Codex",
  cursor: "Cursor"
};
var HOST_SESSION_ENV = {
  "claude-code": "CLAUDE_CODE_SESSION_ID",
  codex: "CODEX_SESSION_ID",
  cursor: null
};
var HOST_SIGN_IN_HINT = {
  "claude-code": "Run /timetrakkr:login to sign in.",
  codex: 'Run "timetrakkr login" in a terminal, or ask Codex to use the timetrakkr-login skill.',
  cursor: 'Run /timetrakkr-login in the agent chat, or "timetrakkr login" in a terminal.'
};
function isHost(value) {
  return HOSTS.some((host) => host === value);
}

// packages/agent-cli/src/private-fs.ts
var import_node_fs = require("node:fs");
var import_node_path = require("node:path");
var DIR_MODE = 448;
var FILE_MODE = 384;
function ensurePrivateDir(dir) {
  (0, import_node_fs.mkdirSync)(dir, { recursive: true, mode: DIR_MODE });
  if (((0, import_node_fs.statSync)(dir).mode & 511) !== DIR_MODE) {
    (0, import_node_fs.chmodSync)(dir, DIR_MODE);
  }
}
function writePrivateJson(path, value) {
  ensurePrivateDir((0, import_node_path.dirname)(path));
  const temporary = `${path}.${process.pid}.tmp`;
  (0, import_node_fs.writeFileSync)(temporary, `${JSON.stringify(value)}
`, { mode: FILE_MODE });
  (0, import_node_fs.renameSync)(temporary, path);
}
function appendPrivateLine(path, line) {
  ensurePrivateDir((0, import_node_path.dirname)(path));
  (0, import_node_fs.appendFileSync)(path, `${line}
`, { mode: FILE_MODE });
}
function readJson(path) {
  try {
    return JSON.parse((0, import_node_fs.readFileSync)(path, "utf8"));
  } catch {
    return null;
  }
}
function isRecord(value) {
  return typeof value === "object" && value !== null && !Array.isArray(value);
}

// packages/agent-cli/src/text.ts
var MAX_LINE = 200;
function oneLine(text, max = MAX_LINE) {
  const flat = text.replace(/[\u0000-\u001f\u007f]+/g, " ").trim();
  return flat.slice(0, max);
}

// packages/agent-cli/src/api.ts
var RETRYABLE_STATUSES = /* @__PURE__ */ new Set([401, 403, 408, 425, 429]);
var MAX_ERROR_LENGTH = 200;
var MEMBER_ID = /^[0-9a-f-]{36}$/i;
async function request(credentials, method, path, signal, body) {
  return fetch(`${credentials.baseUrl}${path}`, {
    method,
    signal,
    // A redirect would hand the token to wherever it points.
    redirect: "error",
    headers: {
      authorization: `Bearer ${credentials.token}`,
      accept: "application/json",
      ...body === void 0 ? {} : { "content-type": "application/json" }
    },
    body: body === void 0 ? void 0 : JSON.stringify(body)
  });
}
async function readJsonBody(response) {
  try {
    return await response.json();
  } catch {
    return null;
  }
}
function failureText(status, body) {
  const message = isRecord(body) && typeof body.message === "string" ? body.message : "";
  const text = message ? `HTTP ${status}: ${message}` : `HTTP ${status}`;
  return text.replace(/\s+/g, " ").slice(0, MAX_ERROR_LENGTH);
}
async function describeFailure(response) {
  return failureText(response.status, await readJsonBody(response));
}
function disabledReason(body) {
  const reason = isRecord(body) ? body.disabled_reason : null;
  const message = isRecord(reason) && typeof reason.message === "string" ? reason.message : "";
  return oneLine(message) || null;
}
function describeThrown(error) {
  if (error instanceof Error && (error.name === "TimeoutError" || error.name === "AbortError")) {
    return "timed out";
  }
  return "no connection";
}
function parseBlocked(value) {
  const { id, remote_sha256: remoteSha256, branch_pattern: branchPattern } = value;
  if (typeof id !== "string" || typeof remoteSha256 !== "string" || !/^[0-9a-f]{64}$/.test(remoteSha256) || branchPattern !== null && typeof branchPattern !== "string") {
    return null;
  }
  return { id, blocked: true, branchPattern, remoteSha256 };
}
function parseMapping(value) {
  if (!isRecord(value)) {
    return null;
  }
  if (value.blocked === true) {
    return parseBlocked(value);
  }
  const {
    id,
    remote,
    branch_pattern: branchPattern,
    project_id: projectId,
    task_id: taskId
  } = value;
  if (typeof id !== "string" || typeof remote !== "string" || typeof projectId !== "string" || typeof taskId !== "string" || branchPattern !== null && typeof branchPattern !== "string") {
    return null;
  }
  const name = (key) => typeof value[key] === "string" ? oneLine(String(value[key])) : "";
  return {
    id,
    remote,
    branchPattern,
    projectId,
    taskId,
    projectName: name("project_name"),
    clientName: name("client_name") || null,
    taskName: name("task_name")
  };
}
function parseMappings(value) {
  return Array.isArray(value) ? value.map(parseMapping).filter((rule) => rule !== null) : [];
}
async function fetchConfig(credentials, signal) {
  try {
    const response = await request(credentials, "GET", "/api/v1/agent/config", signal);
    if (!response.ok) {
      return { ok: false, error: await describeFailure(response), status: response.status };
    }
    const body = await readJsonBody(response);
    if (!isRecord(body) || typeof body.enabled !== "boolean" || !isRecord(body.workspace)) {
      return { ok: false, error: "unexpected response", status: response.status };
    }
    const name = typeof body.workspace.name === "string" ? body.workspace.name : "";
    const memberId = isRecord(body.member) ? body.member.id : null;
    return {
      ok: true,
      enabled: body.enabled,
      workspaceName: name,
      memberId: typeof memberId === "string" && MEMBER_ID.test(memberId) ? memberId : null,
      // Only meaningful while off.
      offReason: body.enabled ? null : disabledReason(body),
      mappings: parseMappings(body.mappings)
    };
  } catch (error) {
    return { ok: false, error: describeThrown(error) };
  }
}
async function putSessionDay(credentials, body, signal) {
  try {
    const response = await request(credentials, "PUT", "/api/v1/agent/session_days", signal, body);
    if (response.ok) {
      return { kind: "ok" };
    }
    const answer = await readJsonBody(response);
    const off = response.status === 403 ? disabledReason(answer) : null;
    if (off) {
      return { kind: "off", reason: off };
    }
    const error = failureText(response.status, answer);
    const retry = response.status >= 500 || RETRYABLE_STATUSES.has(response.status);
    return retry ? { kind: "retry", error, status: response.status } : { kind: "rejected", status: response.status, error };
  } catch (error) {
    return { kind: "retry", error: describeThrown(error) };
  }
}

// packages/agent-cli/src/auth-store.ts
var import_node_crypto = require("node:crypto");
var import_node_fs2 = require("node:fs");

// packages/agent-cli/src/paths.ts
var import_node_os = require("node:os");
var import_node_path2 = require("node:path");
var SAFE_ID = /^[A-Za-z0-9][A-Za-z0-9._-]{0,199}$/;
function isSafeSessionId(id) {
  return typeof id === "string" && SAFE_ID.test(id);
}
function stateHome(env) {
  return env.TIMETRAKKR_HOME || (0, import_node_path2.join)((0, import_node_os.homedir)(), ".timetrakkr");
}
function sessionsDir(home) {
  return (0, import_node_path2.join)(home, "sessions");
}
function hostDir(home, host) {
  return (0, import_node_path2.join)(sessionsDir(home), host);
}
function logPath(home, host, sessionId) {
  return (0, import_node_path2.join)(hostDir(home, host), `${sessionId}.jsonl`);
}
function metaPath(home, host, sessionId) {
  return (0, import_node_path2.join)(hostDir(home, host), `${sessionId}.json`);
}
function tokensPath(home, host, sessionId) {
  return (0, import_node_path2.join)(hostDir(home, host), `${sessionId}.tokens`);
}
function authPath(home) {
  return (0, import_node_path2.join)(home, "auth.json");
}
function loginLockPath(home) {
  return (0, import_node_path2.join)(home, "login.lock");
}
function loginLinkPath(home) {
  return (0, import_node_path2.join)(home, "login-link.json");
}
function signInPromptPath(home) {
  return (0, import_node_path2.join)(home, "sign-in-prompt.json");
}
function authLockPath(home) {
  return (0, import_node_path2.join)(home, "auth.lock");
}
function configCachePath(home) {
  return (0, import_node_path2.join)(home, "config.json");
}
function errorLogPath(home) {
  return (0, import_node_path2.join)(home, "hook.log");
}

// packages/agent-cli/src/auth-store.ts
var STRINGS = [
  "baseUrl",
  "clientId",
  "accessToken",
  "refreshToken",
  "tokenEndpoint",
  "revocationEndpoint"
];
function isSignedOut(state) {
  return "signedOut" in state;
}
function readAuth(home) {
  const value = readJson(authPath(home));
  if (!isRecord(value) || value.version !== 1 || typeof value.baseUrl !== "string") {
    return null;
  }
  if (value.signedOut === true) {
    return { version: 1, baseUrl: value.baseUrl, signedOut: true };
  }
  const complete = STRINGS.every((key) => typeof value[key] === "string" && value[key] !== "") && typeof value.expiresAt === "number";
  return complete ? value : null;
}
function writeAuth(home, state) {
  writePrivateJson(authPath(home), state);
}
function deleteAuth(home) {
  (0, import_node_fs2.rmSync)(authPath(home), { force: true });
}
function fingerprint(token) {
  return (0, import_node_crypto.createHash)("sha256").update(token).digest("hex").slice(0, 16);
}

// packages/agent-cli/src/credentials.ts
var import_node_crypto2 = require("node:crypto");

// packages/agent-cli/src/address.ts
var DEFAULT_WORKSPACE_URL = "https://timetrakkr.com";
function workspaceAddress(env, home, flag) {
  return flag ?? (env.TIMETRAKKR_URL?.trim() || readAuth(home)?.baseUrl || DEFAULT_WORKSPACE_URL);
}

// packages/agent-cli/src/credentials.ts
var LOOPBACK_HOSTS = /* @__PURE__ */ new Set(["localhost", "127.0.0.1", "[::1]"]);
function parseWorkspaceUrl(raw, name) {
  let url;
  try {
    url = new URL(raw);
  } catch {
    return { ok: false, reason: `${name} is not a valid URL` };
  }
  if (url.protocol !== "https:" && url.protocol !== "http:") {
    return { ok: false, reason: `${name} is not a valid URL` };
  }
  if (url.protocol === "http:" && !LOOPBACK_HOSTS.has(url.hostname)) {
    return { ok: false, reason: `${name} must use https` };
  }
  return {
    ok: true,
    baseUrl: `${url.origin}${url.pathname.replace(/\/+$/, "")}`,
    origin: url.origin
  };
}
function digest(text) {
  return (0, import_node_crypto2.createHash)("sha256").update(text).digest("hex").slice(0, 16);
}
function fromEnvironment(token, rawUrl, home) {
  const address = rawUrl ?? readAuth(home)?.baseUrl ?? DEFAULT_WORKSPACE_URL;
  const url = parseWorkspaceUrl(address, "TIMETRAKKR_URL");
  if (!url.ok) {
    return url;
  }
  return {
    ok: true,
    credentials: {
      baseUrl: url.baseUrl,
      token,
      source: "environment",
      cacheId: digest(`${url.baseUrl}
${token}`)
    }
  };
}
function fromSignIn(rawUrl, home) {
  const auth = readAuth(home);
  const wanted = rawUrl ? parseWorkspaceUrl(rawUrl, "TIMETRAKKR_URL") : null;
  if (wanted && !wanted.ok) {
    return wanted;
  }
  if (!auth) {
    const signIn2 = { url: wanted?.origin ?? DEFAULT_WORKSPACE_URL, why: "not signed in" };
    return { ok: false, reason: "not signed in; run timetrakkr login", signIn: signIn2 };
  }
  if (wanted && wanted.origin !== auth.baseUrl) {
    const why = `not signed in to ${wanted.origin}`;
    return {
      ok: false,
      reason: `${why}; run timetrakkr login`,
      signIn: { url: wanted.origin, why }
    };
  }
  if (isSignedOut(auth)) {
    const signIn2 = { url: auth.baseUrl, why: "signed out" };
    return { ok: false, reason: "signed out; run timetrakkr login", signIn: signIn2 };
  }
  const url = parseWorkspaceUrl(auth.baseUrl, "The stored workspace address");
  if (!url.ok) {
    return url;
  }
  return {
    ok: true,
    credentials: {
      baseUrl: url.origin,
      token: auth.accessToken,
      source: "sign-in",
      cacheId: digest(`${url.origin}
${auth.clientId}`),
      expiresAt: auth.expiresAt
    }
  };
}
function readCredentials(env) {
  const home = stateHome(env);
  const token = env.TIMETRAKKR_TOKEN?.trim();
  const rawUrl = env.TIMETRAKKR_URL?.trim() || void 0;
  return token ? fromEnvironment(token, rawUrl, home) : fromSignIn(rawUrl, home);
}

// packages/agent-cli/src/auth-session.ts
var sleep = (ms) => new Promise((done) => setTimeout(done, ms));
function authSession(rt, home, now, signal) {
  let attempted = false;
  async function waitForRenewal(oldToken, rejected, maxWaitMs) {
    attempted = true;
    rt.background(rejected === null ? ["refresh"] : ["refresh", "--rejected", rejected]);
    const giveUpAt = Date.now() + maxWaitMs;
    while (!signal.aborted && Date.now() < giveUpAt) {
      const auth = readAuth(home);
      if (!auth || isSignedOut(auth) || auth.accessToken !== oldToken) {
        return;
      }
      await sleep(RENEW_POLL_MS);
    }
  }
  async function fresh() {
    const current = readCredentials(rt.env);
    if (!current.ok || current.credentials.source !== "sign-in") {
      return current;
    }
    const { expiresAt, token } = current.credentials;
    if (expiresAt === void 0 || expiresAt - now > RENEW_MARGIN_MS) {
      return current;
    }
    const stillValid = expiresAt > now;
    await waitForRenewal(token, null, stillValid ? EARLY_RENEW_WAIT_MS : Number.POSITIVE_INFINITY);
    return readCredentials(rt.env);
  }
  async function renew(rejected) {
    if (rejected.source !== "sign-in") {
      return null;
    }
    const usable = (result) => result.ok && result.credentials.token !== rejected.token ? result.credentials : null;
    const already = usable(readCredentials(rt.env));
    if (already || attempted) {
      return already;
    }
    await waitForRenewal(rejected.token, fingerprint(rejected.token), Number.POSITIVE_INFINITY);
    return usable(readCredentials(rt.env));
  }
  return { fresh, renew };
}

// packages/agent-cli/src/config-cache.ts
var import_node_fs3 = require("node:fs");
function cacheKey(credentials) {
  return credentials.cacheId;
}
function readCache(home, key) {
  const value = readJson(configCachePath(home));
  if (!isRecord(value) || value.key !== key || typeof value.fetchedAt !== "number" || typeof value.enabled !== "boolean" || typeof value.workspaceName !== "string" || !Array.isArray(value.mappings)) {
    return null;
  }
  return {
    key,
    fetchedAt: value.fetchedAt,
    enabled: value.enabled,
    workspaceName: value.workspaceName,
    // Absent from an answer remembered by a version that did not keep it.
    memberId: typeof value.memberId === "string" ? value.memberId : null,
    offReason: typeof value.offReason === "string" ? value.offReason : null,
    // Written by this program from validated rules, in an owner-only file.
    mappings: value.mappings
  };
}
function peekConfig(home, credentials, now) {
  const cached = readCache(home, cacheKey(credentials));
  if (!cached) {
    return null;
  }
  const age = now - cached.fetchedAt;
  return age >= 0 && age < CONFIG_TTL_MS ? {
    ok: true,
    enabled: cached.enabled,
    workspaceName: cached.workspaceName,
    memberId: cached.memberId,
    offReason: cached.offReason,
    mappings: cached.mappings
  } : null;
}
function lastKnownMappings(home, credentials) {
  return readCache(home, cacheKey(credentials))?.mappings ?? null;
}
function lastKnownMemberId(home, credentials) {
  return readCache(home, cacheKey(credentials))?.memberId ?? null;
}
function forgetConfig(home) {
  (0, import_node_fs3.rmSync)(configCachePath(home), { force: true });
}
async function getConfig(home, credentials, now, signal, renew = null) {
  const cached = peekConfig(home, credentials, now);
  if (cached) {
    return cached;
  }
  let fresh = await fetchConfig(credentials, signal);
  if (!fresh.ok && fresh.status === 401 && renew) {
    const renewed = await renew(credentials);
    if (renewed) {
      fresh = await fetchConfig(renewed, signal);
    }
  }
  if (fresh.ok) {
    const entry = {
      key: cacheKey(credentials),
      fetchedAt: now,
      enabled: fresh.enabled,
      workspaceName: fresh.workspaceName,
      memberId: fresh.memberId,
      offReason: fresh.offReason,
      mappings: fresh.mappings
    };
    writePrivateJson(configCachePath(home), entry);
  }
  return fresh;
}

// packages/agent-cli/src/context.ts
var import_node_crypto3 = require("node:crypto");

// packages/core/src/agent-time/turn-clock.ts
var KIND_ORDER = {
  turn_start: 0,
  activity: 1,
  wait: 2,
  turn_end: 3
};
function inTimeOrder(events) {
  return [...events].sort((a, b) => a.at - b.at || KIND_ORDER[a.kind] - KIND_ORDER[b.kind]);
}
function closed(turn, complete) {
  return {
    startedAt: turn.startedAt,
    endedAt: turn.countedTo,
    activeMs: turn.activeMs,
    complete
  };
}
function advance(turn, event) {
  if (event.at < turn.countedTo) {
    return turn;
  }
  if (turn.waitingSince === null) {
    return { ...turn, activeMs: turn.activeMs + (event.at - turn.countedTo), countedTo: event.at };
  }
  const resumedAt = Math.max(turn.waitingSince, event.at - (event.durationMs ?? 0));
  return {
    ...turn,
    activeMs: turn.activeMs + (event.at - resumedAt),
    countedTo: event.at,
    waitingSince: null
  };
}
function turnsFromEvents(events) {
  const turns = [];
  let open = null;
  for (const event of inTimeOrder(events)) {
    if (event.kind === "turn_start") {
      if (open?.startedAt === event.at) {
        continue;
      }
      if (open) {
        turns.push(closed(open, false));
      }
      open = { startedAt: event.at, countedTo: event.at, activeMs: 0, waitingSince: null };
      continue;
    }
    if (!open) {
      continue;
    }
    if (event.kind === "activity") {
      open = advance(open, event);
    } else if (event.kind === "wait") {
      if (open.waitingSince === null) {
        open = { ...advance(open, event), waitingSince: event.at };
      }
    } else {
      const ended = open.waitingSince === null ? advance(open, event) : { ...open, countedTo: Math.max(open.countedTo, event.at) };
      turns.push(closed(ended, true));
      open = null;
    }
  }
  if (open) {
    turns.push(closed(open, false));
  }
  return turns;
}
function summarizeSessionDays(events, dayOf) {
  const byDay = /* @__PURE__ */ new Map();
  for (const turn of turnsFromEvents(events)) {
    if (turn.activeMs <= 0) {
      continue;
    }
    const day = dayOf(turn.startedAt);
    byDay.set(day, [...byDay.get(day) ?? [], turn]);
  }
  return [...byDay.entries()].sort(([a], [b]) => a.localeCompare(b)).map(([day, turns]) => ({
    day,
    activeSeconds: Math.round(turns.reduce((sum, t) => sum + t.activeMs, 0) / 1e3),
    turnCount: turns.length,
    firstTurnStartedAt: Math.min(...turns.map((t) => t.startedAt)),
    lastTurnEndedAt: Math.max(...turns.map((t) => t.endedAt))
  }));
}

// packages/core/src/agent-time/repository-mapping.ts
var SCP_FORM = /^(?:[^@/\s]+@)?([^:/\s]+):(?!\/\/)(.+)$/;
var URL_FORM = /^[a-z][a-z0-9+.-]*:\/\/(?:[^@/\s]+@)?([^:/\s]+)(?::\d+)?\/(.+)$/i;
var BARE_FORM = /^([a-z0-9.-]+\.[a-z]{2,})\/(.+)$/i;
function hostAndPath(remote) {
  if (/^file:/i.test(remote)) {
    return null;
  }
  const match = URL_FORM.exec(remote) ?? SCP_FORM.exec(remote) ?? BARE_FORM.exec(remote);
  const host = match?.[1];
  const path = match?.[2];
  return host && path ? [host, path] : null;
}
function normalizeRemote(remote) {
  const parts = hostAndPath(remote.trim().replace(/[?#].*$/, ""));
  if (!parts) {
    return null;
  }
  const [host, rawPath] = parts;
  const path = rawPath.replace(/\/+$/, "").replace(/\.git$/i, "").replace(/^\/+/, "");
  if (!path.includes("/") || /\s/.test(path) || !host.includes(".")) {
    return null;
  }
  return `${host}/${path}`.toLowerCase();
}
function branchMatches(pattern, branch) {
  const parts = pattern.split("*");
  const first = parts[0] ?? "";
  if (parts.length === 1) {
    return branch === first;
  }
  const last = parts[parts.length - 1] ?? "";
  if (branch.length < first.length + last.length || !branch.startsWith(first) || !branch.endsWith(last)) {
    return false;
  }
  const end = branch.length - last.length;
  let from = first.length;
  for (const part of parts.slice(1, -1)) {
    const at = branch.indexOf(part, from);
    if (at === -1 || at + part.length > end) {
      return false;
    }
    from = at + part.length;
  }
  return true;
}
function specificity(pattern) {
  return pattern.replace(/\*/g, "").length;
}
function resolveMapping(mappings, remote, branch) {
  const wanted = remote === null ? null : normalizeRemote(remote);
  if (wanted === null) {
    return null;
  }
  const forRepository = mappings.filter((m) => normalizeRemote(m.remote) === wanted);
  const byBranch = forRepository.filter(
    (m) => m.branchPattern !== null && branch !== null && branchMatches(m.branchPattern, branch)
  ).sort((a, b) => specificity(b.branchPattern) - specificity(a.branchPattern));
  return byBranch[0] ?? forRepository.find((m) => m.branchPattern === null) ?? null;
}

// packages/core/src/agent-time/model-prices.ts
var usd = (model, input, output, cacheRead, cacheWrite) => ({ model, currency: "USD", input, output, cacheRead, cacheWrite });
var DEFAULT_MODEL_PRICES = [
  usd("claude-fable-5-1", "10", "50", "0.25", "12.50"),
  usd("claude-opus-5-5", "4", "20", "0.20", "5"),
  usd("claude-opus-5", "5", "25", "0.50", "6.25"),
  usd("claude-opus-4-8", "5", "25", "0.50", "6.25"),
  usd("claude-opus-4-7", "5", "25", "0.50", "6.25"),
  usd("claude-opus-4-6", "5", "25", "0.50", "6.25"),
  usd("claude-sonnet-5-5", "2", "10", "0.20", "2.50"),
  usd("claude-sonnet-5", "2", "10", "0.20", "2.50"),
  usd("claude-sonnet-4-6", "3", "15", "0.30", "3.75"),
  usd("claude-sonnet-4-5", "3", "15", "0.30", "3.75"),
  usd("claude-haiku-4-5", "1", "5", "0.10", "1.25"),
  usd("gpt-6-astra", "10", "50", "1.00", "12.50"),
  usd("gpt-6.1-sol", "2", "10", "0.10", "2.50"),
  usd("gpt-6-luna", "0.10", "0.50", "0.01", "0.125"),
  usd("gpt-5.6-sol", "4", "20", "0.40", "5.00"),
  usd("gpt-5.3-codex", "1.75", "14", "0.175", null)
];

// packages/core/src/agent-time/untrusted-text.ts
var FORMAT = new RegExp("\\p{Cf}", "gu");
var CONTROL = new RegExp("\\p{Cc}", "gu");

// packages/agent-cli/src/git.ts
var import_node_child_process = require("node:child_process");
function git(root, env, args) {
  try {
    const out = (0, import_node_child_process.execFileSync)("git", ["-C", root, ...args], {
      encoding: "utf8",
      timeout: GIT_TIMEOUT_MS,
      stdio: ["ignore", "pipe", "ignore"],
      env: { ...env, GIT_TERMINAL_PROMPT: "0" }
    });
    return out.trim() || null;
  } catch {
    return null;
  }
}
function withoutQuery(remote) {
  return remote.replace(/[?#].*$/s, "");
}
function repositoryRemote(root, env) {
  const raw = git(root, env, ["config", "--get", "remote.origin.url"]);
  const remote = raw === null ? null : withoutQuery(raw);
  return remote !== null && remote.length <= MAX_REPOSITORY_LENGTH ? remote : null;
}
function currentBranch(root, env) {
  const branch = git(root, env, ["symbolic-ref", "--short", "-q", "HEAD"]);
  return branch !== null && branch.length <= MAX_BRANCH_LENGTH ? branch : null;
}

// packages/agent-cli/src/mapping.ts
var import_node_fs4 = require("node:fs");
var import_node_path3 = require("node:path");
var UUID = /^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$/i;
function findRepositoryRoot(cwd) {
  let dir = cwd;
  for (; ; ) {
    if ((0, import_node_fs4.existsSync)((0, import_node_path3.join)(dir, ".git"))) {
      return dir;
    }
    const parent = (0, import_node_path3.dirname)(dir);
    if (parent === dir) {
      return null;
    }
    dir = parent;
  }
}
function readMappingText(path) {
  try {
    if ((0, import_node_fs4.statSync)(path).size > MAPPING_MAX_BYTES) {
      return null;
    }
    return (0, import_node_fs4.readFileSync)(path, "utf8");
  } catch {
    return null;
  }
}
function parseMapping2(text) {
  let value;
  try {
    value = JSON.parse(text);
  } catch {
    return { reason: `${MAPPING_FILE} is not valid JSON` };
  }
  if (!isRecord(value)) {
    return { reason: `${MAPPING_FILE} must be an object with projectId and taskId` };
  }
  const { projectId, taskId } = value;
  if (typeof projectId !== "string" || !UUID.test(projectId)) {
    return { reason: `${MAPPING_FILE}: projectId must be a UUID` };
  }
  if (typeof taskId !== "string" || !UUID.test(taskId)) {
    return { reason: `${MAPPING_FILE}: taskId must be a UUID` };
  }
  return { mapping: { projectId: projectId.toLowerCase(), taskId: taskId.toLowerCase() } };
}
function readMapping(root) {
  const path = (0, import_node_path3.join)(root, MAPPING_FILE);
  if (!(0, import_node_fs4.existsSync)(path)) {
    return { missing: true };
  }
  const text = readMappingText(path);
  return text === null ? { reason: `${MAPPING_FILE} could not be read` } : parseMapping2(text);
}

// packages/agent-cli/src/context.ts
var NOT_MAPPED = "this repository is not mapped to a project";
var BLOCKED = "this branch is mapped to a project you cannot log time to";
function readLocalContext(cwd, env) {
  const root = findRepositoryRoot(cwd);
  if (root === null) {
    return { root, file: null, remote: null, branch: null };
  }
  const file = readMapping(root);
  if ("reason" in file) {
    return { root, file, remote: null, branch: null };
  }
  return { root, file, remote: repositoryRemote(root, env), branch: currentBranch(root, env) };
}
function couldBeTracked(local) {
  if (local.file === null || "reason" in local.file) {
    return false;
  }
  return "mapping" in local.file || local.remote !== null && normalizeRemote(local.remote) !== null;
}
function labelOf(rule) {
  return [rule.clientName, rule.projectName, rule.taskName].filter(Boolean).join(" / ");
}
function untracked(root, untrackedReason) {
  return { root, mapping: null, untrackedReason, repository: null, branch: null };
}
function candidatesFor(rules, remote) {
  const own = normalizeRemote(remote);
  if (own === null) {
    return [];
  }
  const digest2 = (0, import_node_crypto3.createHash)("sha256").update(own).digest("hex");
  return rules.flatMap((rule) => {
    if (!("blocked" in rule)) {
      return [rule];
    }
    return rule.remoteSha256 === digest2 ? [
      {
        id: rule.id,
        remote: own,
        branchPattern: rule.branchPattern,
        projectId: "",
        taskId: "",
        blocked: true
      }
    ] : [];
  });
}
function fromRules(local, rules, credentials) {
  const { root, remote, branch } = local;
  if (remote === null || normalizeRemote(remote) === null) {
    return untracked(root, `no ${MAPPING_FILE} and no git remote to look up`);
  }
  if (!credentials.ok) {
    return untracked(root, credentials.reason);
  }
  if (rules === null) {
    return untracked(root, "could not reach Time Trakkr to look up this repository");
  }
  const rule = resolveMapping(candidatesFor(rules, remote), remote, branch);
  if (rule === null) {
    return untracked(root, NOT_MAPPED);
  }
  if ("blocked" in rule) {
    return untracked(root, BLOCKED);
  }
  return {
    root,
    mapping: {
      projectId: rule.projectId,
      taskId: rule.taskId,
      mappingId: rule.id,
      source: "platform",
      label: labelOf(rule) || null
    },
    untrackedReason: null,
    repository: normalizeRemote(remote),
    branch
  };
}
function resolveContext(local, rules, credentials) {
  const { root, file } = local;
  if (root === null || file === null) {
    return untracked(root, "not inside a git repository");
  }
  if ("reason" in file) {
    return untracked(root, file.reason);
  }
  if ("missing" in file) {
    return fromRules(local, rules, credentials);
  }
  const { projectId, taskId } = file.mapping;
  const named = rules?.find(
    (rule) => !("blocked" in rule) && rule.projectId === projectId && rule.taskId === taskId
  );
  return {
    root,
    mapping: {
      projectId,
      taskId,
      mappingId: null,
      source: "file",
      label: named ? labelOf(named) || null : null
    },
    untrackedReason: null,
    // Null for a remote that is not a hosted repository, such as a local path.
    repository: local.remote === null ? null : normalizeRemote(local.remote),
    branch: local.branch
  };
}

// packages/agent-cli/src/day.ts
var pad = (n) => String(n).padStart(2, "0");
function localDay(at) {
  const date = new Date(at);
  return `${date.getFullYear()}-${pad(date.getMonth() + 1)}-${pad(date.getDate())}`;
}

// packages/agent-cli/src/drop.ts
var import_node_fs7 = require("node:fs");

// packages/agent-cli/src/identity.ts
var import_node_crypto4 = require("node:crypto");
var MEMBER_PREFIX = "member:";
function memberIdentity(credentials, memberId) {
  const { origin } = new URL(credentials.baseUrl);
  const digest2 = (0, import_node_crypto4.createHash)("sha256").update(`${origin}
${memberId}`).digest("hex").slice(0, 16);
  return `${MEMBER_PREFIX}${digest2}`;
}
function isMemberBound(identity) {
  return identity.startsWith(MEMBER_PREFIX);
}
function memberIsKnown(home, credentials) {
  return lastKnownMemberId(home, credentials) !== null;
}
function identitiesOf(home, credentials) {
  const memberId = lastKnownMemberId(home, credentials);
  return memberId === null ? [credentials.cacheId] : [memberIdentity(credentials, memberId), credentials.cacheId];
}
function ownedBy(session, identities) {
  const { identity, credentialId } = session;
  return typeof identity === "string" && identities.includes(identity) || typeof credentialId === "string" && identities.includes(credentialId);
}

// packages/agent-cli/src/store.ts
var import_node_fs5 = require("node:fs");
var EVENT_KINDS = ["turn_start", "activity", "wait", "turn_end"];
function parseEvent(line) {
  try {
    const value = JSON.parse(line);
    if (!isRecord(value) || typeof value.at !== "number") {
      return null;
    }
    const kind = EVENT_KINDS.find((k) => k === value.kind);
    if (!kind) {
      return null;
    }
    return typeof value.durationMs === "number" ? { kind, at: value.at, durationMs: value.durationMs } : { kind, at: value.at };
  } catch {
    return null;
  }
}
function appendEvent(home, host, sessionId, event) {
  appendPrivateLine(logPath(home, host, sessionId), JSON.stringify(event));
}
function readLog(home, host, sessionId) {
  let raw;
  try {
    raw = (0, import_node_fs5.readFileSync)(logPath(home, host, sessionId));
  } catch {
    return { events: [], bytes: 0 };
  }
  const events = raw.toString("utf8").split("\n").map(parseEvent).filter((event) => event !== null);
  return { events, bytes: raw.length };
}
function logSize(home, host, sessionId) {
  try {
    return (0, import_node_fs5.statSync)(logPath(home, host, sessionId)).size;
  } catch {
    return 0;
  }
}
function isMeta(value, host, sessionId) {
  return isRecord(value) && value.version === 1 && value.host === host && value.sessionId === sessionId && isRecord(value.posted);
}
function readMeta(home, host, sessionId) {
  const value = readJson(metaPath(home, host, sessionId));
  return isMeta(value, host, sessionId) ? value : null;
}
function writeMeta(home, meta) {
  writePrivateJson(metaPath(home, meta.host, meta.sessionId), meta);
}
function sessionIdsOf(home, host) {
  try {
    return (0, import_node_fs5.readdirSync)(hostDir(home, host)).filter((name) => name.endsWith(".json")).map((name) => name.slice(0, -".json".length)).filter(isSafeSessionId);
  } catch {
    return [];
  }
}
function listMetas(home) {
  return HOSTS.flatMap(
    (host) => sessionIdsOf(home, host).map((sessionId) => readMeta(home, host, sessionId)).filter((meta) => meta !== null)
  );
}
function pruneSessions(home, before) {
  for (const meta of listMetas(home)) {
    if (meta.updatedAt < before) {
      (0, import_node_fs5.rmSync)(metaPath(home, meta.host, meta.sessionId), { force: true });
      (0, import_node_fs5.rmSync)(logPath(home, meta.host, meta.sessionId), { force: true });
      (0, import_node_fs5.rmSync)(tokensPath(home, meta.host, meta.sessionId), { force: true });
    }
  }
}

// packages/agent-cli/src/hosts/transcript.ts
var import_node_fs6 = require("node:fs");
var NEWLINE = 10;
function countOf(value, optional) {
  if (value === void 0 && optional) {
    return 0;
  }
  return typeof value === "number" && Number.isInteger(value) && value >= 0 ? value : null;
}
function readWholeLines(path, offset) {
  const stat = (0, import_node_fs6.statSync)(path);
  if (!stat.isFile() || stat.size < offset) {
    return null;
  }
  const length = Math.min(stat.size - offset, TRANSCRIPT_READ_BYTES);
  const buffer = Buffer.alloc(length);
  const fd = (0, import_node_fs6.openSync)(path, "r");
  try {
    (0, import_node_fs6.readSync)(fd, buffer, 0, length, offset);
  } finally {
    (0, import_node_fs6.closeSync)(fd);
  }
  const end = buffer.lastIndexOf(NEWLINE);
  if (end === -1) {
    return length === TRANSCRIPT_READ_BYTES ? null : { lines: [], offset };
  }
  const lines = buffer.subarray(0, end).toString("utf8").split("\n").filter((line) => line.trim() !== "");
  return { lines, offset: offset + end + 1 };
}

// packages/agent-cli/src/hosts/claude-code-transcript.ts
function parseUsage(usage) {
  const input = countOf(usage.input_tokens, false);
  const output = countOf(usage.output_tokens, false);
  const cacheRead = countOf(usage.cache_read_input_tokens, true);
  const cacheWrite = countOf(usage.cache_creation_input_tokens, true);
  if (input === null || output === null || cacheRead === null || cacheWrite === null) {
    return null;
  }
  return { input, output, cache_read: cacheRead, cache_write: cacheWrite };
}
function parseLine(line) {
  let row;
  try {
    row = JSON.parse(line);
  } catch {
    return null;
  }
  if (!isRecord(row)) {
    return null;
  }
  if (row.type !== "assistant" || !isRecord(row.message) || !isRecord(row.message.usage)) {
    return "skip";
  }
  const { id, model, usage } = row.message;
  if (typeof model === "string" && model.startsWith("<")) {
    return "skip";
  }
  const at = typeof row.timestamp === "string" ? Date.parse(row.timestamp) : Number.NaN;
  const counts = isRecord(usage) ? parseUsage(usage) : null;
  if (typeof id !== "string" || id.length === 0 || typeof model !== "string" || model.length === 0 || model.length > MAX_MODEL_LENGTH || Number.isNaN(at) || counts === null) {
    return null;
  }
  return { messageId: id, model, at, counts };
}
function readClaudeCodeUsage(path, offset) {
  try {
    const chunk = readWholeLines(path, offset);
    if (chunk === null) {
      return { ok: false };
    }
    const entries = [];
    for (const line of chunk.lines) {
      const parsed = parseLine(line);
      if (parsed === null) {
        return { ok: false };
      }
      if (parsed !== "skip") {
        entries.push(parsed);
      }
    }
    return { ok: true, entries, offset: chunk.offset };
  } catch {
    return { ok: false };
  }
}

// packages/agent-cli/src/hosts/signal.ts
function durationOf(value) {
  return typeof value === "number" && Number.isFinite(value) && value >= 0 ? value : void 0;
}
function modelOf(value) {
  return typeof value === "string" && value.length > 0 && value.length <= MAX_MODEL_LENGTH ? value : null;
}
function pathOf(value) {
  return typeof value === "string" && value.length > 0 ? value : void 0;
}

// packages/agent-cli/src/hosts/codex-rollout.ts
function parseUsage2(usage) {
  const allInput = countOf(usage.input_tokens, false);
  const output = countOf(usage.output_tokens, false);
  const cacheRead = countOf(usage.cached_input_tokens, true);
  const cacheWrite = countOf(usage.cache_write_input_tokens, true);
  if (allInput === null || output === null || cacheRead === null || cacheWrite === null) {
    return null;
  }
  if (cacheRead > allInput) {
    return null;
  }
  return { input: allInput - cacheRead, output, cache_read: cacheRead, cache_write: cacheWrite };
}
function parseRecord(row, model) {
  const { payload } = row;
  const at = typeof row.timestamp === "string" ? Date.parse(row.timestamp) : Number.NaN;
  if (!isRecord(payload) || !isRecord(payload.usage) || model === null || Number.isNaN(at)) {
    return null;
  }
  const counts = parseUsage2(payload.usage);
  const id = payload.response_id;
  if (typeof id !== "string" || id.length === 0 || counts === null) {
    return null;
  }
  return { messageId: id, model, at, counts };
}
function readCodexUsage(path, offset, seedModel) {
  try {
    const chunk = readWholeLines(path, offset);
    if (chunk === null) {
      return { ok: false };
    }
    const entries = [];
    let model = seedModel;
    for (const line of chunk.lines) {
      const row = JSON.parse(line);
      if (!isRecord(row)) {
        return { ok: false };
      }
      if (row.type === "turn_context" && isRecord(row.payload)) {
        model = modelOf(row.payload.model) ?? model;
      }
      if (row.type === "token_usage_record") {
        const entry = parseRecord(row, model);
        if (entry === null) {
          return { ok: false };
        }
        entries.push(entry);
      }
    }
    return { ok: true, entries, offset: chunk.offset };
  } catch {
    return { ok: false };
  }
}

// packages/agent-cli/src/tokens.ts
var READERS = {
  "claude-code": readClaudeCodeUsage,
  codex: readCodexUsage,
  cursor: null
};
var EMPTY = {
  version: 1,
  offset: 0,
  unreliable: false,
  lastModel: null,
  days: {},
  recent: []
};
function readTokenState(home, host, sessionId) {
  const value = readJson(tokensPath(home, host, sessionId));
  return isRecord(value) && value.version === 1 && isRecord(value.days) && Array.isArray(value.recent) ? value : EMPTY;
}
function writeTokenState(home, host, sessionId, state) {
  writePrivateJson(tokensPath(home, host, sessionId), state);
}
function markTokensUnreliable(home, host, sessionId) {
  const state = readTokenState(home, host, sessionId);
  writeTokenState(home, host, sessionId, { ...state, unreliable: true });
}
function add(a, b, sign) {
  return {
    input: a.input + sign * b.input,
    output: a.output + sign * b.output,
    cache_read: a.cache_read + sign * b.cache_read,
    cache_write: a.cache_write + sign * b.cache_write
  };
}
var ZERO = { input: 0, output: 0, cache_read: 0, cache_write: 0 };
function adjust(days, message, sign) {
  const day = days[message.day] ?? {};
  const next = add(day[message.model] ?? ZERO, message.counts, sign);
  return { ...days, [message.day]: { ...day, [message.model]: next } };
}
function dayOfReply(events, at, dayOf) {
  const turns = turnsFromEvents(events);
  const index = turns.findLastIndex((turn2) => turn2.startedAt <= at);
  const turn = turns[index];
  if (!turn) {
    return null;
  }
  const next = turns[index + 1];
  const until = turn.complete ? turn.endedAt + REPLY_SLACK_MS : next?.startedAt ?? Number.POSITIVE_INFINITY;
  return at <= until ? dayOf(turn.startedAt) : null;
}
function count(state, entry, day) {
  const earlier = state.recent.find((message2) => message2.id === entry.messageId);
  const withoutEarlier = earlier ? adjust(state.days, earlier, -1) : state.days;
  const recent = state.recent.filter((message2) => message2.id !== entry.messageId);
  if (day === null) {
    return { ...state, days: withoutEarlier, recent };
  }
  const message = {
    id: entry.messageId,
    day,
    model: entry.model,
    counts: entry.counts
  };
  return {
    ...state,
    lastModel: entry.model,
    days: adjust(withoutEarlier, message, 1),
    recent: [...recent, message].slice(-RECENT_MESSAGES)
  };
}
function updateTokens(home, host, sessionId, transcriptPath, events, dayOf, hookModel) {
  const state = readTokenState(home, host, sessionId);
  const reader = READERS[host];
  if (state.unreliable || reader === null) {
    return state;
  }
  try {
    const read = reader(transcriptPath, state.offset, hookModel ?? state.lastModel);
    const next = read.ok ? read.entries.reduce(
      (acc, entry) => count(acc, entry, dayOfReply(events, entry.at, dayOf)),
      { ...state, offset: read.offset }
    ) : { ...state, unreliable: true };
    writeTokenState(home, host, sessionId, next);
    return next;
  } catch {
    return { ...state, unreliable: true };
  }
}
function tokensForDay(state, day) {
  const totals = state.unreliable ? void 0 : state.days[day];
  if (!totals) {
    return null;
  }
  const models = Object.entries(totals).filter(([, c]) => totalOf({ model: c }) > 0);
  return models.length > 0 && models.length <= MAX_TOKEN_MODELS ? Object.fromEntries(models) : null;
}
function totalOf(totals) {
  return Object.values(totals ?? {}).reduce(
    (sum, c) => sum + c.input + c.output + c.cache_read + c.cache_write,
    0
  );
}

// packages/agent-cli/src/drop.ts
function dropPending(home, meta) {
  const { host, sessionId } = meta;
  const size = logSize(home, host, sessionId);
  const unsentEstimate = meta.estimate && !meta.estimate.sent;
  if (size === meta.syncedBytes && !meta.unsent && !unsentEstimate) {
    return;
  }
  if (size > meta.syncedBytes) {
    (0, import_node_fs7.truncateSync)(logPath(home, host, sessionId), meta.syncedBytes);
    markTokensUnreliable(home, host, sessionId);
  }
  const last = readLog(home, host, sessionId).events.at(-1);
  if (last && last.kind !== "turn_end") {
    appendEvent(home, host, sessionId, { kind: "turn_end", at: last.at });
  }
  const latest = readMeta(home, host, sessionId) ?? meta;
  writeMeta(home, {
    ...latest,
    estimate: unsentEstimate ? null : latest.estimate,
    syncedBytes: logSize(home, host, sessionId),
    unsent: false,
    lastError: null
  });
}
function dropAllPending(home, identities) {
  for (const meta of listMetas(home)) {
    if (ownedBy(meta, identities)) {
      dropPending(home, meta);
    }
  }
}

// packages/agent-cli/src/flush.ts
function toPosted(summary, item) {
  const tokenTotal = totalOf(tokensForDay(item.tokens, summary.day));
  return {
    activeSeconds: summary.activeSeconds,
    turnCount: summary.turnCount,
    firstTurnStartedAt: summary.firstTurnStartedAt,
    lastTurnEndedAt: summary.lastTurnEndedAt,
    projectId: item.mapping.projectId,
    taskId: item.mapping.taskId,
    ...tokenTotal > 0 ? { tokenTotal } : {}
  };
}
function alreadyPosted(posted, next) {
  return posted !== void 0 && posted.activeSeconds === next.activeSeconds && posted.turnCount === next.turnCount && posted.firstTurnStartedAt === next.firstTurnStartedAt && posted.lastTurnEndedAt === next.lastTurnEndedAt && posted.projectId === next.projectId && posted.taskId === next.taskId && (posted.tokenTotal ?? 0) === (next.tokenTotal ?? 0);
}
function toBody(item, summary, withEstimate) {
  const { meta, mapping } = item;
  const tokens = tokensForDay(item.tokens, summary.day);
  return {
    host: meta.host,
    session_id: meta.sessionId,
    day: summary.day,
    active_seconds: Math.min(summary.activeSeconds, SECONDS_PER_DAY, windowSeconds(summary)),
    turn_count: Math.min(summary.turnCount, MAX_TURN_COUNT),
    first_turn_started_at: new Date(summary.firstTurnStartedAt).toISOString(),
    last_turn_ended_at: new Date(summary.lastTurnEndedAt).toISOString(),
    project_id: mapping.projectId,
    task_id: mapping.taskId,
    repository: meta.repository,
    branch: meta.branch,
    // Nothing safe to summarize from yet: the prompt is never read.
    summary: null,
    model: meta.model ?? item.tokens.lastModel,
    mapping_id: mapping.mappingId ?? null,
    // Left out, not zeroed, when the transcript could not be read.
    ...tokens ? { tokens } : {},
    ...withEstimate && item.estimate ? { estimated_effort: { seconds: item.estimate.seconds, label: item.estimate.label } } : {}
  };
}
function needsAttention(home, meta, now, isOwn) {
  if (isOwn || meta.unsent) {
    return true;
  }
  if (logSize(home, meta.host, meta.sessionId) === meta.syncedBytes) {
    return false;
  }
  const last = readLog(home, meta.host, meta.sessionId).events.at(-1);
  return last !== void 0 && last.at <= now - STALE_SESSION_MS;
}
function windowSeconds(summary) {
  const window = Math.floor((summary.lastTurnEndedAt - summary.firstTurnStartedAt) / 1e3);
  return Math.max(0, window) + WINDOW_SLACK_SECONDS;
}
function stillAccepted(day, now) {
  const oldest = new Date(now - MAX_DAY_AGE_DAYS * SECONDS_PER_DAY * 1e3);
  return day >= oldest.toISOString().slice(0, 10);
}
function workFor(home, meta, mapping, now) {
  const log = readLog(home, meta.host, meta.sessionId);
  const tokens = readTokenState(home, meta.host, meta.sessionId);
  const estimate = meta.estimate && !meta.estimate.sent ? meta.estimate : null;
  const summaries = summarizeSessionDays(log.events, localDay).filter(
    (summary) => stillAccepted(summary.day, now)
  );
  const changed = summaries.filter(
    (summary) => !alreadyPosted(
      meta.posted[summary.day],
      toPosted(summary, { meta, mapping, tokens, estimate })
    )
  );
  const latest = summaries.at(-1);
  const pending = changed.length === 0 && estimate && latest ? [latest] : changed;
  return { meta, mapping, tokens, bytes: log.bytes, pending, estimate };
}
function collectWork(options, identities) {
  const { home, now, own } = options;
  const work = [];
  for (const meta of listMetas(home)) {
    const { mapping } = meta;
    if (!ownedBy(meta, identities)) {
      continue;
    }
    if (!mapping || meta.off || meta.updatedAt < now - RETRY_WINDOW_MS) {
      continue;
    }
    const isOwn = own?.host === meta.host && own.sessionId === meta.sessionId;
    if (needsAttention(home, meta, now, isOwn)) {
      work.push(workFor(home, meta, mapping, now));
    }
  }
  return work;
}
function patchMeta(home, key, patch, sentEstimate = null) {
  const latest = readMeta(home, key.host, key.sessionId) ?? key;
  const estimate = sentEstimate && latest.estimate?.at === sentEstimate.at ? { ...latest.estimate, sent: true } : latest.estimate;
  writeMeta(home, {
    ...latest,
    ...patch,
    estimate,
    posted: { ...latest.posted, ...patch.posted }
  });
}
function holdBack(home, work, error) {
  for (const { meta } of work) {
    if (!meta.unsent || meta.lastError !== error) {
      patchMeta(home, meta, { unsent: true, lastError: error });
    }
  }
}
function settle(home, work) {
  for (const { meta, bytes } of work) {
    if (meta.unsent || meta.syncedBytes !== bytes) {
      patchMeta(home, meta, { unsent: false, syncedBytes: bytes });
    }
  }
}
async function postSession(home, credentials, item, signal, renew) {
  let current = credentials;
  const posted = {};
  let failure = null;
  let rejection = null;
  let sentEstimate = null;
  for (const summary of item.pending) {
    const withEstimate = summary === item.pending.at(-1);
    const body = toBody(item, summary, withEstimate);
    let result = await putSessionDay(current, body, signal);
    if (result.kind === "retry" && result.status === 401 && renew) {
      const renewed = await renew(current);
      if (renewed) {
        current = renewed;
        result = await putSessionDay(current, body, signal);
      }
    }
    if (result.kind === "off") {
      return { credentials: current, off: true };
    }
    if (result.kind === "retry") {
      failure = result.error;
      break;
    }
    const acknowledged = toPosted(summary, item);
    if (result.kind === "rejected") {
      rejection = result.error;
      posted[summary.day] = { ...acknowledged, rejected: result.status };
    } else {
      posted[summary.day] = acknowledged;
    }
    if (withEstimate) {
      sentEstimate = item.estimate;
    }
  }
  patchMeta(
    home,
    item.meta,
    failure === null ? { posted, unsent: false, syncedBytes: item.bytes, lastError: rejection } : { posted, unsent: true, lastError: failure },
    sentEstimate
  );
  return { credentials: current, off: false };
}
async function flush(options) {
  const { home, env, now, signal } = options;
  const credentials = readCredentials(env);
  if (!credentials.ok) {
    return { config: null };
  }
  const identities = identitiesOf(home, credentials.credentials);
  const work = collectWork(options, identities);
  const sendable = work.filter((item) => item.pending.length > 0);
  const config = options.config ?? (sendable.length > 0 ? await getConfig(home, credentials.credentials, now, signal, options.renew ?? null) : null);
  if (config?.ok && !config.enabled) {
    dropAllPending(home, identities);
    return { config };
  }
  settle(
    home,
    work.filter((item) => item.pending.length === 0)
  );
  if (!config || sendable.length === 0) {
    return { config };
  }
  if (!config.ok) {
    holdBack(home, sendable, config.error);
    return { config };
  }
  let current = credentials.credentials;
  for (const item of sendable) {
    if (signal.aborted) {
      holdBack(home, [item], "timed out");
      continue;
    }
    const posted = await postSession(home, current, item, signal, options.renew ?? null);
    if (posted.off) {
      forgetConfig(home);
      dropAllPending(home, identities);
      return { config };
    }
    current = posted.credentials;
  }
  return { config };
}

// packages/agent-cli/src/hosts/claude-code.ts
function mapClaudeCodeHook(event, payload) {
  if (!isRecord(payload)) {
    return null;
  }
  const { session_id: sessionId, cwd } = payload;
  if (!isSafeSessionId(sessionId) || typeof cwd !== "string" || cwd.length === 0) {
    return null;
  }
  const signal = (kind) => ({
    type: "event",
    sessionId,
    cwd,
    event: { kind }
  });
  const transcriptPath = pathOf(payload.transcript_path);
  const transcript = transcriptPath ? { transcriptPath } : {};
  switch (event) {
    case "SessionStart":
      return {
        type: "session_start",
        sessionId,
        cwd,
        // Present on some starts only; a headless start sends none.
        model: modelOf(payload.model),
        // A compaction restarts the session under the person's feet, mid-work.
        announce: payload.source !== "compact"
      };
    case "UserPromptSubmit":
      return signal("turn_start");
    case "PreToolUse":
      return signal("activity");
    case "PostToolUse": {
      const durationMs = durationOf(payload.duration_ms);
      return durationMs === void 0 ? signal("activity") : { type: "event", sessionId, cwd, event: { kind: "activity", durationMs } };
    }
    case "Notification":
      return payload.notification_type === "permission_prompt" ? signal("wait") : null;
    case "Stop":
      return { type: "event", sessionId, cwd, event: { kind: "turn_end" }, ...transcript };
    case "SessionEnd":
      return { type: "session_end", sessionId, cwd, ...transcript };
    default:
      return null;
  }
}

// packages/agent-cli/src/hosts/codex.ts
function mapCodexHook(event, payload) {
  if (!isRecord(payload)) {
    return null;
  }
  const { session_id: sessionId, cwd } = payload;
  if (!isSafeSessionId(sessionId) || typeof cwd !== "string" || cwd.length === 0) {
    return null;
  }
  const model = modelOf(payload.model);
  const transcriptPath = pathOf(payload.transcript_path);
  const signal = (kind, withTranscript = false) => ({
    type: "event",
    sessionId,
    cwd,
    event: { kind },
    ...model ? { model } : {},
    ...withTranscript && transcriptPath ? { transcriptPath } : {}
  });
  switch (event) {
    case "SessionStart":
      return {
        type: "session_start",
        sessionId,
        cwd,
        model,
        // A compaction restarts the session under the person's feet, mid-work.
        announce: payload.source !== "compact"
      };
    case "UserPromptSubmit":
      return signal("turn_start");
    case "PreToolUse":
    case "PostToolUse":
      return signal("activity");
    case "Stop":
    case "Interrupt":
      return signal("turn_end", true);
    case "SessionEnd":
      return { type: "session_end", sessionId, cwd, ...transcriptPath ? { transcriptPath } : {} };
    default:
      return null;
  }
}

// packages/agent-cli/src/hosts/cursor.ts
function directoryOf(payload) {
  if (typeof payload.cwd === "string" && payload.cwd.length > 0) {
    return payload.cwd;
  }
  const roots = payload.workspace_roots;
  const first = Array.isArray(roots) ? roots[0] : void 0;
  return typeof first === "string" && first.length > 0 ? first : null;
}
function mapCursorHook(event, payload) {
  if (!isRecord(payload)) {
    return null;
  }
  const sessionId = payload.conversation_id ?? payload.session_id;
  const cwd = directoryOf(payload);
  if (!isSafeSessionId(sessionId) || cwd === null) {
    return null;
  }
  const model = modelOf(payload.model);
  const signal = (kind, durationMs) => ({
    type: "event",
    sessionId,
    cwd,
    event: durationMs === void 0 ? { kind } : { kind, durationMs },
    ...model ? { model } : {}
  });
  switch (event) {
    case "sessionStart":
      return { type: "session_start", sessionId, cwd, model, announce: false };
    case "beforeSubmitPrompt":
      return signal("turn_start");
    case "preToolUse":
    case "postToolUseFailure":
      return signal("activity");
    case "postToolUse":
      return signal("activity", durationOf(payload.duration));
    case "stop":
      return signal("turn_end");
    case "sessionEnd":
      return { type: "session_end", sessionId, cwd };
    default:
      return null;
  }
}

// packages/agent-cli/src/login-background.ts
var import_node_fs8 = require("node:fs");
function lockAge(home, now) {
  const value = readJson(loginLockPath(home));
  return isRecord(value) && typeof value.at === "number" ? now - value.at : null;
}
function loginInProgress(home, now) {
  const age = lockAge(home, now);
  return age !== null && age < LOGIN_LOCK_STALE_MS;
}
function takeLoginLock(home, now) {
  ensurePrivateDir(home);
  const path = loginLockPath(home);
  for (let attempt = 0; attempt < 2; attempt += 1) {
    try {
      const fd = (0, import_node_fs8.openSync)(path, "wx", 384);
      (0, import_node_fs8.writeSync)(fd, JSON.stringify({ at: now }));
      (0, import_node_fs8.closeSync)(fd);
      return () => (0, import_node_fs8.rmSync)(path, { force: true });
    } catch {
      if (loginInProgress(home, now)) {
        return null;
      }
      (0, import_node_fs8.rmSync)(path, { force: true });
    }
  }
  return null;
}
function readLoginLink(home) {
  const value = readJson(loginLinkPath(home));
  if (isRecord(value) && typeof value.url === "string") {
    return { url: value.url };
  }
  return isRecord(value) && typeof value.error === "string" ? { error: value.error } : null;
}
function writeLoginLink(home, link) {
  writePrivateJson(loginLinkPath(home), link);
}
function clearLoginLink(home) {
  (0, import_node_fs8.rmSync)(loginLinkPath(home), { force: true });
}
async function waitForLoginLink(home, now) {
  const deadline = Date.now() + LOGIN_LINK_WAIT_MS;
  let started = false;
  while (Date.now() < deadline) {
    const link = readLoginLink(home);
    if (link) {
      return link;
    }
    const running = loginInProgress(home, now);
    if (started && !running) {
      return null;
    }
    started = started || running;
    await new Promise((done) => setTimeout(done, LOCK_POLL_MS));
  }
  return null;
}
function quiet(rt, errors) {
  return { ...rt, out: () => void 0, err: (text) => errors.push(text) };
}

// packages/agent-cli/src/sign-in-prompt.ts
function readRecord(home) {
  const value = readJson(signInPromptPath(home));
  return {
    lastOpenedDay: isRecord(value) && typeof value.lastOpenedDay === "string" ? value.lastOpenedDay : null,
    declined: isRecord(value) && value.declined === true
  };
}
function setDeclined(home, declined) {
  writePrivateJson(signInPromptPath(home), { ...readRecord(home), declined });
}
function promptSignIn(rt, home, at, credentials, couldTrack) {
  if (credentials.ok || !credentials.signIn || !couldTrack) {
    return null;
  }
  const record = readRecord(home);
  const today = localDay(at);
  if (record.declined || record.lastOpenedDay === today || loginInProgress(home, at)) {
    return "not-now";
  }
  writePrivateJson(signInPromptPath(home), { ...record, lastOpenedDay: today });
  rt.background(["login", "--child"]);
  return "opened";
}

// packages/agent-cli/src/status-line.ts
var NAME = "Time Trakkr";
var MAX_WORKSPACE_NAME = 80;
function notTracked(reason) {
  return `${NAME}: not tracked (${reason}).`;
}
function workspaceOf(config) {
  const name = config?.ok ? oneLine(config.workspaceName, MAX_WORKSPACE_NAME) : "";
  return name || "this workspace";
}
function switchedOffLine(config) {
  const reason = config?.ok ? config.offReason : null;
  return reason ? `${NAME}: not tracked. ${reason}` : notTracked(`agent time is not switched on for ${workspaceOf(config)}`);
}
function describeMapping(mapping) {
  return mapping.label ?? `project ${mapping.projectId}, task ${mapping.taskId}`;
}
function signInLine(host, why, prompt) {
  return prompt === "opened" ? `${NAME}: ${why}. A browser window has opened to sign in; once you allow it, tracking starts at your next prompt.` : `${notTracked(why)} ${HOST_SIGN_IN_HINT[host]}`;
}
function sessionStartLine(meta, credentials, config) {
  const switchedOff = config?.ok === true && !config.enabled;
  if (!meta.mapping) {
    return switchedOff && meta.untrackedReason === NOT_MAPPED ? switchedOffLine(config) : notTracked(meta.untrackedReason ?? "no mapping");
  }
  if (!credentials.ok) {
    return notTracked(credentials.reason);
  }
  if (meta.off) {
    return `${NAME}: tracking is off for this session. Run "timetrakkr on" to resume.`;
  }
  if (switchedOff) {
    return switchedOffLine(config);
  }
  const target = describeMapping(meta.mapping);
  if (config && !config.ok) {
    const problem = config.status === void 0 ? `could not reach ${NAME} (${config.error})` : `sign-in failed (${oneLine(config.error)})`;
    return `${NAME}: tracking to ${target}, but ${problem}. Nothing is sent until that works.`;
  }
  return `${NAME}: tracking to ${target}. Run "timetrakkr off" to stop.`;
}

// packages/agent-cli/src/hook.ts
var ADAPTERS = {
  "claude-code": mapClaudeCodeHook,
  codex: mapCodexHook,
  cursor: mapCursorHook
};
function modelFrom(signal) {
  if (signal.type === "session_start") {
    return signal.model;
  }
  return signal.type === "event" ? signal.model ?? null : null;
}
function parsePayload(text) {
  try {
    return JSON.parse(text);
  } catch {
    return null;
  }
}
function newMeta(host, sessionId, cwd) {
  return {
    version: 1,
    host,
    sessionId,
    cwd,
    root: null,
    repository: null,
    branch: null,
    mapping: null,
    untrackedReason: null,
    off: false,
    model: null,
    estimate: null,
    posted: {},
    syncedBytes: 0,
    unsent: false,
    lastError: null,
    updatedAt: 0
  };
}
function refreshesContext(signal) {
  return signal.type === "session_start" || signal.type === "event" && signal.event.kind === "turn_start";
}
function reachesServer(signal) {
  return signal.type !== "event" || signal.event.kind === "turn_start" || signal.event.kind === "turn_end";
}
var FOREIGN_SESSION = "this session began under a different sign-in; start a new session to track it";
function isTrackable(home, meta, credentials) {
  return meta.mapping !== null && !meta.off && credentials.ok && ownedBy(meta, identitiesOf(home, credentials.credentials));
}
async function isForeign(run2, existing) {
  const { home, credentials, auth, at, signal } = run2;
  if (!credentials.ok || typeof existing.identity !== "string") {
    return false;
  }
  const mine = () => ownedBy(existing, identitiesOf(home, credentials.credentials));
  if (mine()) {
    return false;
  }
  if (!isMemberBound(existing.identity) || memberIsKnown(home, credentials.credentials)) {
    return true;
  }
  await getConfig(home, credentials.credentials, at, signal, auth.renew);
  return !mine();
}
async function refresh(run2, existing) {
  const { rt, home, host, hookSignal, credentials, auth, at, signal } = run2;
  if (existing && await isForeign(run2, existing)) {
    const meta2 = { ...existing, mapping: null, untrackedReason: FOREIGN_SESSION };
    return { meta: meta2, config: null, couldTrack: false };
  }
  const local = readLocalContext(hookSignal.cwd, rt.env);
  const couldTrack = couldBeTracked(local);
  const config = credentials.ok && couldTrack ? await getConfig(home, credentials.credentials, at, signal, auth.renew) : null;
  const rules = config?.ok ? config.mappings : credentials.ok ? lastKnownMappings(home, credentials.credentials) : null;
  const context = resolveContext(local, rules, credentials);
  if (existing?.mapping && !context.mapping) {
    await flush({
      home,
      env: rt.env,
      now: at,
      signal,
      own: existing,
      config,
      renew: auth.renew
    });
  }
  const base = readMeta(home, host, hookSignal.sessionId) ?? newMeta(host, hookSignal.sessionId, "");
  const model = modelFrom(hookSignal) ?? base.model;
  const tracked = context.mapping !== null;
  const meta = {
    ...base,
    ...context,
    // Where an untracked session is running is not written down.
    cwd: tracked ? hookSignal.cwd : base.cwd,
    root: tracked ? context.root : base.root,
    // The member, once the server has said who that is; the credentials until then.
    identity: tracked && credentials.ok ? identitiesOf(home, credentials.credentials)[0] ?? null : base.identity ?? null,
    credentialId: tracked && credentials.ok ? credentials.credentials.cacheId : base.credentialId ?? null,
    model,
    updatedAt: at
  };
  if (tracked || existing) {
    writeMeta(home, meta);
  }
  return { meta, config, couldTrack };
}
async function load(run2) {
  const { home, host, hookSignal, credentials, at } = run2;
  const existing = readMeta(home, host, hookSignal.sessionId);
  if (refreshesContext(hookSignal)) {
    return refresh(run2, existing);
  }
  if (!existing) {
    return { meta: newMeta(host, hookSignal.sessionId, ""), config: null, couldTrack: false };
  }
  const config = credentials.ok && isTrackable(home, existing, credentials) ? peekConfig(home, credentials.credentials, at) : null;
  return { meta: existing, config, couldTrack: false };
}
function readTokens(run2, meta) {
  const { home, host, hookSignal } = run2;
  if (hookSignal.type === "session_start" || !hookSignal.transcriptPath) {
    return;
  }
  const { events } = readLog(home, host, meta.sessionId);
  updateTokens(
    home,
    host,
    meta.sessionId,
    hookSignal.transcriptPath,
    events,
    localDay,
    modelFrom(hookSignal) ?? meta.model
  );
}
async function handleHook(rt, host, event, signal = AbortSignal.timeout(NETWORK_BUDGET_MS)) {
  const at = rt.now();
  const hookSignal = ADAPTERS[host](event, parsePayload(await rt.stdin()));
  if (!hookSignal) {
    return;
  }
  const home = stateHome(rt.env);
  const auth = authSession(rt, home, at, signal);
  const credentials = reachesServer(hookSignal) ? await auth.fresh() : readCredentials(rt.env);
  const run2 = { rt, home, host, hookSignal, credentials, auth, at, signal };
  const { meta, config, couldTrack } = await load(run2);
  const switchedOff = config?.ok === true && !config.enabled;
  const recording = isTrackable(home, meta, credentials) && !switchedOff;
  if (hookSignal.type === "event" && recording) {
    appendEvent(home, host, meta.sessionId, { ...hookSignal.event, at });
  }
  if (!reachesServer(hookSignal)) {
    return;
  }
  if (recording) {
    readTokens(run2, meta);
  }
  const prompt = hookSignal.type === "session_start" ? promptSignIn(rt, home, at, credentials, couldTrack) : null;
  if (hookSignal.type === "session_start") {
    pruneSessions(home, at - PRUNE_AFTER_MS);
  }
  const outcome = await flush({
    home,
    env: rt.env,
    now: at,
    signal,
    own: { host, sessionId: meta.sessionId },
    config,
    renew: auth.renew
  });
  if (hookSignal.type === "session_start" && hookSignal.announce) {
    const line = prompt && !credentials.ok && credentials.signIn ? signInLine(host, credentials.signIn.why, prompt) : sessionStartLine(meta, credentials, outcome.config);
    rt.out(JSON.stringify({ systemMessage: line }));
  }
}

// packages/agent-cli/src/session-choice.ts
var NO_SESSION = "No session found for this directory. Run this from inside the agent session.";
function sessionsFromEnvironment(rt, home) {
  return HOSTS.flatMap((host) => {
    const variable = HOST_SESSION_ENV[host];
    const sessionId = variable ? rt.env[variable] : void 0;
    return variable && isSafeSessionId(sessionId) ? [{ host, sessionId, because: `from ${variable}`, meta: readMeta(home, host, sessionId) }] : [];
  });
}
function sessionFromEnvironment(rt, home) {
  const named = sessionsFromEnvironment(rt, home);
  const active = named.filter((session) => session.meta !== null).sort((a, b) => (b.meta?.updatedAt ?? 0) - (a.meta?.updatedAt ?? 0));
  return active[0] ?? named[0] ?? null;
}
function mostRecentSessionHere(rt, home) {
  const root = findRepositoryRoot(rt.cwd);
  const here = listMetas(home).filter((meta2) => meta2.cwd === rt.cwd || root !== null && meta2.root === root).sort((a, b) => b.updatedAt - a.updatedAt);
  const meta = here[0];
  return meta ? {
    host: meta.host,
    sessionId: meta.sessionId,
    because: "the most recent session in this directory; the host did not name one",
    meta
  } : null;
}
function chooseSession(rt, home) {
  return sessionFromEnvironment(rt, home) ?? mostRecentSessionHere(rt, home);
}

// packages/agent-cli/src/commands.ts
function switchTracking(rt, off) {
  const home = stateHome(rt.env);
  const chosen = chooseSession(rt, home);
  if (!chosen) {
    rt.err(NO_SESSION);
    return 1;
  }
  const meta = chosen.meta ?? newMeta(chosen.host, chosen.sessionId, "");
  if (off && !meta.off && meta.mapping) {
    appendEvent(home, meta.host, meta.sessionId, { kind: "turn_end", at: rt.now() });
  }
  writeMeta(home, { ...meta, off, updatedAt: rt.now() });
  rt.out(
    `Tracking is ${off ? "off" : "on"} for ${HOST_LABELS[chosen.host]} session ${chosen.sessionId} (${chosen.because}).`
  );
  return 0;
}
function sourceNote(credentials, home) {
  if (credentials.source === "sign-in") {
    return "signed in with timetrakkr login";
  }
  const stored = readAuth(home);
  return stored && !isSignedOut(stored) ? "TIMETRAKKR_TOKEN from the environment; the stored sign-in is not used" : "TIMETRAKKR_TOKEN from the environment";
}
function signInLines(credentials, config, home) {
  if (!credentials.ok) {
    return [`Sign-in:    ${credentials.reason}`];
  }
  if (!config) {
    return ["Sign-in:    not checked (nothing here could be tracked)"];
  }
  if (!config.ok) {
    return [`Sign-in:    failed (${oneLine(config.error)})`];
  }
  const workspace = oneLine(config.workspaceName) || "workspace";
  return [
    `Sign-in:    OK (${sourceNote(credentials.credentials, home)})`,
    `Workspace:  ${workspace}, agent time is ${config.enabled ? "on" : "off"}`,
    ...config.offReason ? [`Why off:    ${config.offReason}`] : []
  ];
}
function mappingLines(context) {
  const { mapping } = context;
  if (!mapping) {
    return [`Not tracked: ${context.untrackedReason ?? "no mapping"}`];
  }
  const source = mapping.source === "file" ? ".timetrakkr.json" : "a rule on the platform";
  return [
    `Tracked to: ${describeMapping(mapping)} (${source})`,
    `Project:    ${mapping.projectId}`,
    `Task:       ${mapping.taskId}`
  ];
}
async function askConfig(credentials, renew, signal) {
  const first = await fetchConfig(credentials, signal);
  if (first.ok || first.status !== 401) {
    return first;
  }
  const renewed = await renew(credentials);
  return renewed ? fetchConfig(renewed, signal) : first;
}
async function printStatus(rt) {
  const home = stateHome(rt.env);
  const signal = AbortSignal.timeout(COMMAND_NETWORK_MS);
  const auth = authSession(rt, home, rt.now(), signal);
  const credentials = await auth.fresh();
  const local = readLocalContext(rt.cwd, rt.env);
  const config = credentials.ok && couldBeTracked(local) ? await askConfig(credentials.credentials, auth.renew, signal) : null;
  const rules = config?.ok ? config.mappings : credentials.ok ? lastKnownMappings(home, credentials.credentials) : null;
  const context = resolveContext(local, rules, credentials);
  const lines = [`Directory:  ${rt.cwd}`];
  if (context.root) {
    lines.push(`Repository: ${context.root}`);
  }
  lines.push(...mappingLines(context), ...signInLines(credentials, config, home));
  const session = chooseSession(rt, home);
  if (session?.meta) {
    const state = session.meta.off ? "tracking off" : "tracking on";
    lines.push(`Session:    ${session.sessionId}, ${state} (${session.because})`);
    if (session.meta.lastError) {
      lines.push(`Last error: ${oneLine(session.meta.lastError)}`);
    }
  }
  for (const line of lines) {
    rt.out(line);
  }
  return 0;
}

// packages/agent-cli/src/error-log.ts
var import_node_fs9 = require("node:fs");
function logHookError(home, now, where, error) {
  try {
    const path = errorLogPath(home);
    const message = error instanceof Error ? error.message : "unexpected error";
    try {
      if ((0, import_node_fs9.statSync)(path).size > ERROR_LOG_MAX_BYTES) {
        (0, import_node_fs9.truncateSync)(path);
      }
    } catch {
    }
    appendPrivateLine(path, `${new Date(now).toISOString()} ${where}: ${message}`);
  } catch {
  }
}

// packages/agent-cli/src/estimate.ts
var UNITS = [
  { flag: "--weeks", word: "week", hours: DAYS_PER_WEEK * HOURS_PER_DAY },
  { flag: "--days", word: "day", hours: HOURS_PER_DAY },
  { flag: "--hours", word: "hour", hours: 1 }
];
var FLAGS = [...UNITS.map((unit) => unit.flag), "--label"];
function readFlags(args) {
  const flags = /* @__PURE__ */ new Map();
  for (let i = 0; i < args.length; i += 1) {
    const [name = "", inline] = (args[i] ?? "").split(/=(.*)/s, 2);
    if (!FLAGS.includes(name)) {
      return `Unknown option ${name}.`;
    }
    const value = inline ?? args[++i];
    if (value === void 0) {
      return `${name} needs a value.`;
    }
    flags.set(name, value);
  }
  return flags;
}
function parseEstimate(args) {
  const flags = readFlags(args);
  if (typeof flags === "string") {
    return { ok: false, error: flags };
  }
  let hours = 0;
  const words = [];
  for (const unit of UNITS) {
    const raw = flags.get(unit.flag);
    if (raw === void 0) {
      continue;
    }
    const amount = Number(raw);
    if (raw.trim() === "" || !Number.isFinite(amount)) {
      return { ok: false, error: `${unit.flag} must be a number.` };
    }
    if (amount <= 0) {
      return { ok: false, error: `${unit.flag} must be greater than zero.` };
    }
    hours += amount * unit.hours;
    words.push(`${amount} ${unit.word}${amount === 1 ? "" : "s"}`);
  }
  if (words.length === 0) {
    return { ok: false, error: "Give a duration: --hours, --days or --weeks." };
  }
  const seconds = Math.round(hours * 3600);
  if (seconds < 1 || seconds > MAX_ESTIMATE_SECONDS) {
    return {
      ok: false,
      error: `The estimate must be at most ${MAX_ESTIMATE_SECONDS / 3600} hours.`
    };
  }
  const label = oneLine(flags.get("--label") ?? "", Number.MAX_SAFE_INTEGER);
  if (label === "") {
    return { ok: false, error: "Give a short name for the work with --label." };
  }
  if (label.length > MAX_ESTIMATE_LABEL) {
    return { ok: false, error: `--label must be at most ${MAX_ESTIMATE_LABEL} characters.` };
  }
  return { ok: true, seconds, label, words: words.join(" ") };
}
function recordEstimate(rt, args) {
  const parsed = parseEstimate(args);
  if (!parsed.ok) {
    rt.err(parsed.error);
    return 1;
  }
  const home = stateHome(rt.env);
  const chosen = chooseSession(rt, home);
  if (!chosen) {
    rt.err(NO_SESSION);
    return 1;
  }
  const { meta } = chosen;
  if (!meta?.mapping || meta.off) {
    rt.out("This session is not tracked, so the estimate was not recorded.");
    return 0;
  }
  const estimate = { seconds: parsed.seconds, label: parsed.label, at: rt.now(), sent: false };
  writeMeta(home, { ...meta, estimate });
  rt.out(
    `Recorded an estimate of ${parsed.words} ("${parsed.label}") for ${HOST_LABELS[chosen.host]} session ${chosen.sessionId}.`
  );
  return 0;
}

// packages/agent-cli/src/login.ts
var import_node_crypto5 = require("node:crypto");
var import_node_os2 = require("node:os");

// packages/agent-cli/src/login-listener.ts
var import_node_http = require("node:http");
var STYLE = "body{font:16px/1.5 system-ui,sans-serif;max-width:32rem;margin:15vh auto;padding:0 1.5rem;color:#1a1a1a}h1{font-size:1.4rem}";
var PAGE = (title, body) => `<!doctype html><html lang="en"><head><meta charset="utf-8"><title>${title}</title><meta name="viewport" content="width=device-width, initial-scale=1"><style>${STYLE}</style></head><body><h1>${title}</h1><p>${body}</p></body></html>`;
var PAGES = {
  signedIn: PAGE(
    "You are signed in",
    "Time Trakkr is connected on this computer. You can close this tab and go back to your terminal."
  ),
  failed: PAGE(
    "Sign-in did not finish",
    "Time Trakkr could not complete the sign-in. Go back to your terminal for the reason, then try again."
  ),
  denied: PAGE(
    "Sign-in was declined",
    "Nothing was connected. You can close this tab. Run the sign-in again if that was a mistake."
  ),
  mismatch: PAGE(
    "This link is not for this sign-in",
    "It does not match the sign-in waiting in your terminal. Go back to the terminal and use the link shown there."
  ),
  notFound: PAGE("Nothing here", "This address only finishes a Time Trakkr sign-in.")
};
function respond(res, status, page) {
  res.writeHead(status, {
    "content-type": "text/html; charset=utf-8",
    "cache-control": "no-store",
    "referrer-policy": "no-referrer",
    "x-content-type-options": "nosniff",
    "content-security-policy": "default-src 'none'; style-src 'unsafe-inline'",
    connection: "close"
  });
  res.end(page);
}
function closeServer(server) {
  return new Promise((done) => {
    server.closeAllConnections();
    server.close(() => done());
  });
}
async function startLoginListener() {
  let handler = (_req, res) => respond(res, 404, PAGES.notFound);
  const server = (0, import_node_http.createServer)((req, res) => handler(req, res));
  await new Promise((done, fail) => {
    server.once("error", fail);
    server.listen(0, "127.0.0.1", done);
  });
  const { port } = server.address();
  const redirectUri = `http://127.0.0.1:${port}/callback`;
  const wait = (state, timeoutMs, settle2) => new Promise((done) => {
    let finished = false;
    const finish = (outcome) => {
      finished = true;
      clearTimeout(timer);
      done(outcome);
    };
    const timer = setTimeout(() => finish({ kind: "timeout" }), timeoutMs);
    handler = (req, res) => {
      const url = new URL(req.url ?? "/", redirectUri);
      if (finished || req.method !== "GET" || url.pathname !== "/callback") {
        return respond(res, 404, PAGES.notFound);
      }
      if (url.searchParams.get("state") !== state) {
        return respond(res, 400, PAGES.mismatch);
      }
      const code = url.searchParams.get("code");
      if (!code) {
        respond(res, 200, PAGES.denied);
        return finish({ kind: "denied" });
      }
      finished = true;
      settle2(code).catch(() => "failed").then((verdict) => {
        respond(res, 200, verdict === "signed-in" ? PAGES.signedIn : PAGES.failed);
        finish({ kind: "code", code });
      });
    };
  });
  return { redirectUri, wait, close: () => closeServer(server) };
}

// packages/agent-cli/src/oauth.ts
var FORM = { "content-type": "application/x-www-form-urlencoded", accept: "application/json" };
async function readJsonBody2(response) {
  try {
    return await response.json();
  } catch {
    return null;
  }
}
function describeThrown2(error) {
  const timedOut = error instanceof Error && (error.name === "TimeoutError" || error.name === "AbortError");
  return timedOut ? "timed out" : "no connection";
}
async function discover(origin, signal) {
  let body;
  try {
    const response = await fetch(`${origin}/.well-known/oauth-authorization-server`, {
      signal,
      redirect: "error",
      headers: { accept: "application/json" }
    });
    if (!response.ok) {
      return { ok: false, error: `the server answered HTTP ${response.status}` };
    }
    body = await readJsonBody2(response);
  } catch (error) {
    return { ok: false, error: describeThrown2(error) };
  }
  if (!isRecord(body)) {
    return { ok: false, error: "the server did not describe its sign-in endpoints" };
  }
  const endpoints = {
    authorize: body.authorization_endpoint,
    token: body.token_endpoint,
    register: body.registration_endpoint,
    revoke: body.revocation_endpoint
  };
  for (const url of Object.values(endpoints)) {
    if (typeof url !== "string") {
      return { ok: false, error: "the server did not describe its sign-in endpoints" };
    }
    if (!URL.canParse(url) || new URL(url).origin !== origin) {
      return { ok: false, error: "the server pointed sign-in at another host" };
    }
  }
  return { ok: true, endpoints };
}
async function registerClient(endpoint, clientName, redirectUri, signal) {
  try {
    const response = await fetch(endpoint, {
      method: "POST",
      signal,
      redirect: "error",
      headers: { "content-type": "application/json", accept: "application/json" },
      body: JSON.stringify({
        client_name: clientName,
        redirect_uris: [redirectUri],
        token_endpoint_auth_method: "none"
      })
    });
    const body = await readJsonBody2(response);
    if (response.ok && isRecord(body) && typeof body.client_id === "string" && body.client_id) {
      return { ok: true, clientId: body.client_id };
    }
    const code = isRecord(body) && typeof body.error === "string" ? `: ${body.error}` : "";
    return { ok: false, error: `the server answered HTTP ${response.status}${code}` };
  } catch (error) {
    return { ok: false, error: describeThrown2(error) };
  }
}
function parsePair(body) {
  if (!isRecord(body) || typeof body.access_token !== "string" || typeof body.refresh_token !== "string" || typeof body.expires_in !== "number" || body.access_token === "" || body.refresh_token === "" || !(body.expires_in > 0)) {
    return { ok: false, kind: "unavailable", error: "unexpected response" };
  }
  if (body.scope !== "agent") {
    return { ok: false, kind: "wrong_scope" };
  }
  return {
    ok: true,
    pair: {
      accessToken: body.access_token,
      refreshToken: body.refresh_token,
      expiresIn: body.expires_in
    }
  };
}
async function tokenRequest(endpoint, form, signal) {
  try {
    const response = await fetch(endpoint, {
      method: "POST",
      signal,
      redirect: "error",
      headers: FORM,
      body: new URLSearchParams(form).toString()
    });
    const body = await readJsonBody2(response);
    if (response.ok) {
      return parsePair(body);
    }
    const code = isRecord(body) && typeof body.error === "string" ? body.error : "";
    return code === "invalid_grant" || code === "invalid_client" ? { ok: false, kind: "invalid_grant" } : { ok: false, kind: "unavailable", error: `HTTP ${response.status}` };
  } catch (error) {
    return { ok: false, kind: "unavailable", error: describeThrown2(error) };
  }
}
function exchangeCode(endpoint, grant, signal) {
  return tokenRequest(
    endpoint,
    {
      grant_type: "authorization_code",
      code: grant.code,
      client_id: grant.clientId,
      redirect_uri: grant.redirectUri,
      code_verifier: grant.codeVerifier
    },
    signal
  );
}
function refreshPair(endpoint, clientId, refreshToken, signal) {
  return tokenRequest(
    endpoint,
    { grant_type: "refresh_token", refresh_token: refreshToken, client_id: clientId },
    signal
  );
}
async function revoke(endpoint, clientId, refreshToken, signal) {
  try {
    const response = await fetch(endpoint, {
      method: "POST",
      signal,
      redirect: "error",
      headers: FORM,
      body: new URLSearchParams({ token: refreshToken, client_id: clientId }).toString()
    });
    return response.ok ? { ok: true } : { ok: false, error: `HTTP ${response.status}` };
  } catch (error) {
    return { ok: false, error: describeThrown2(error) };
  }
}

// packages/agent-cli/src/login.ts
var MAX_CLIENT_NAME = 200;
var MODE_FLAGS = { "--detach": "detach", "--child": "child" };
function parseOptions(args) {
  let url;
  let timeoutMs = LOGIN_TIMEOUT_MS;
  let browser = true;
  let mode = "interactive";
  for (let i = 0; i < args.length; i += 1) {
    const [name = "", inline] = (args[i] ?? "").split(/=(.*)/s, 2);
    if (name === "--no-browser") {
      browser = false;
      continue;
    }
    const flagMode = MODE_FLAGS[name];
    if (flagMode) {
      mode = flagMode;
      continue;
    }
    if (name !== "--url" && name !== "--timeout") {
      return { ok: false, error: `Unknown option ${name}.` };
    }
    const value = inline ?? args[++i];
    if (value === void 0) {
      return { ok: false, error: `${name} needs a value.` };
    }
    if (name === "--url") {
      url = value;
    } else {
      const seconds = Number(value);
      if (!Number.isFinite(seconds) || seconds <= 0) {
        return { ok: false, error: "--timeout must be a number of seconds." };
      }
      timeoutMs = seconds * 1e3;
    }
  }
  return { ok: true, url, timeoutMs, browser, mode };
}
function authorizeUrl(endpoints, clientId, redirectUri, state, codeVerifier) {
  const url = new URL(endpoints.authorize);
  url.search = new URLSearchParams({
    response_type: "code",
    client_id: clientId,
    redirect_uri: redirectUri,
    scope: "agent",
    state,
    code_challenge: (0, import_node_crypto5.createHash)("sha256").update(codeVerifier).digest("base64url"),
    code_challenge_method: "S256"
  }).toString();
  return url.toString();
}
function describeTokenFailure(result) {
  if (result.kind === "wrong_scope") {
    return "The server granted a scope other than agent time, so the sign-in was not kept.";
  }
  if (result.kind === "invalid_grant") {
    return "The server refused the sign-in code. Run timetrakkr login again.";
  }
  return `The sign-in could not be completed (${result.error}). Run timetrakkr login again.`;
}
async function welcomeLine(rt) {
  const credentials = readCredentials({
    ...rt.env,
    TIMETRAKKR_TOKEN: void 0,
    TIMETRAKKR_URL: void 0
  });
  if (!credentials.ok) {
    return "Signed in.";
  }
  const config = await getConfig(
    stateHome(rt.env),
    credentials.credentials,
    rt.now(),
    AbortSignal.timeout(COMMAND_NETWORK_MS)
  );
  if (!config.ok) {
    return "Signed in.";
  }
  const workspace = oneLine(config.workspaceName, 80) || "your workspace";
  return config.enabled ? `Signed in to ${workspace}. Agent time is on for that workspace.` : `Signed in to ${workspace}. Agent time is not switched on for that workspace, so nothing is tracked yet.`;
}
async function signIn(rt, options, onLink = () => void 0) {
  const home = stateHome(rt.env);
  const flagName = options.url ? "--url" : "TIMETRAKKR_URL";
  const workspace = parseWorkspaceUrl(workspaceAddress(rt.env, home, options.url), flagName);
  if (!workspace.ok) {
    rt.err(`${workspace.reason}.`);
    return 1;
  }
  const { origin } = workspace;
  const setup = AbortSignal.timeout(COMMAND_NETWORK_MS);
  const discovered = await discover(origin, setup);
  if (!discovered.ok) {
    rt.err(`Could not start sign-in with ${origin}: ${discovered.error}.`);
    return 1;
  }
  const { endpoints } = discovered;
  const listener = await startLoginListener();
  try {
    const clientName = `Time Trakkr CLI on ${(0, import_node_os2.hostname)()}`.slice(0, MAX_CLIENT_NAME);
    const client = await registerClient(endpoints.register, clientName, listener.redirectUri, setup);
    if (!client.ok) {
      rt.err(`Could not start sign-in with ${origin}: ${client.error}.`);
      return 1;
    }
    const state = (0, import_node_crypto5.randomBytes)(24).toString("base64url");
    const codeVerifier = (0, import_node_crypto5.randomBytes)(48).toString("base64url");
    const url = authorizeUrl(endpoints, client.clientId, listener.redirectUri, state, codeVerifier);
    onLink(url);
    rt.out(
      options.browser ? "Opening your browser to sign in. If it does not open, go to:" : "Open this link in a browser to sign in:"
    );
    rt.out(url);
    if (options.browser) {
      rt.openBrowser(url);
    }
    const exchange = { failure: null };
    const outcome = await listener.wait(
      state,
      options.timeoutMs,
      async (code) => {
        const result = await exchangeCode(
          endpoints.token,
          { code, clientId: client.clientId, redirectUri: listener.redirectUri, codeVerifier },
          AbortSignal.timeout(COMMAND_NETWORK_MS)
        );
        if (!result.ok) {
          exchange.failure = describeTokenFailure(result);
          return "failed";
        }
        writeAuth(home, {
          version: 1,
          baseUrl: origin,
          clientId: client.clientId,
          accessToken: result.pair.accessToken,
          refreshToken: result.pair.refreshToken,
          expiresAt: rt.now() + result.pair.expiresIn * 1e3,
          tokenEndpoint: endpoints.token,
          revocationEndpoint: endpoints.revoke
        });
        return "signed-in";
      }
    );
    if (outcome.kind === "timeout") {
      rt.err(
        "Sign-in was not completed in time. If the browser showed an error, or said agent time is not switched on for the workspace, that is why. Run timetrakkr login to try again."
      );
      return 1;
    }
    if (outcome.kind === "denied") {
      rt.err("Sign-in was declined in the browser. Nothing was connected.");
      return 1;
    }
    if (exchange.failure !== null) {
      rt.err(exchange.failure);
      return 1;
    }
    setDeclined(home, false);
    if (rt.env.TIMETRAKKR_TOKEN?.trim()) {
      rt.out("Note: TIMETRAKKR_TOKEN is set and takes precedence over this sign-in.");
    }
    rt.out(await welcomeLine(rt));
    return 0;
  } finally {
    await listener.close();
  }
}
async function childSignIn(rt, options) {
  const home = stateHome(rt.env);
  const release = takeLoginLock(home, rt.now());
  if (!release) {
    return 0;
  }
  const errors = [];
  let linked = false;
  try {
    const code = await signIn(quiet(rt, errors), options, (url) => {
      linked = true;
      writeLoginLink(home, { url });
    });
    if (!linked && code !== 0) {
      writeLoginLink(home, { error: errors[0] ?? "the sign-in stopped" });
    }
    return code;
  } finally {
    release();
  }
}
function printLink(rt, url) {
  rt.out("Opening your browser to sign in. If it does not open, go to:");
  rt.out(url);
  rt.out("Come back here once you have allowed it. Tracking starts at your next prompt.");
}
async function detachedSignIn(rt, url) {
  const home = stateHome(rt.env);
  const waiting = loginInProgress(home, rt.now()) ? readLoginLink(home) : null;
  if (waiting && "url" in waiting) {
    rt.openBrowser(waiting.url);
    printLink(rt, waiting.url);
    return 0;
  }
  clearLoginLink(home);
  rt.background(["login", "--child", ...url ? ["--url", url] : []]);
  const link = await waitForLoginLink(home, rt.now());
  if (!link || "error" in link) {
    const why = link ? ` (${oneLine(link.error)})` : "";
    rt.err(`The sign-in could not be started${why}. Run "timetrakkr login" in a terminal to retry.`);
    return 1;
  }
  printLink(rt, link.url);
  return 0;
}
async function login(rt, args) {
  const options = parseOptions(args);
  if (!options.ok) {
    rt.err(options.error);
    return 1;
  }
  if (options.mode === "detach") {
    return detachedSignIn(rt, options.url);
  }
  return options.mode === "child" ? childSignIn(rt, options) : signIn(rt, options);
}
async function logout(rt) {
  const home = stateHome(rt.env);
  setDeclined(home, true);
  const auth = readAuth(home);
  const environmentNote = rt.env.TIMETRAKKR_TOKEN?.trim() ? " TIMETRAKKR_TOKEN is still set in the environment and will keep being used." : "";
  if (!auth) {
    rt.out(`Not signed in.${environmentNote}`);
    return 0;
  }
  const revoked = isSignedOut(auth) ? null : await revoke(
    auth.revocationEndpoint,
    auth.clientId,
    auth.refreshToken,
    AbortSignal.timeout(COMMAND_NETWORK_MS)
  );
  deleteAuth(home);
  if (revoked === null) {
    rt.out(`Signed out on this machine.${environmentNote}`);
  } else if (revoked.ok) {
    rt.out(`Signed out. The sign-in was revoked on the server.${environmentNote}`);
  } else {
    rt.out(
      `Signed out on this machine. Revoking it on the server could not be confirmed (${revoked.error}); you can remove it under Developer \u2192 Connected apps.${environmentNote}`
    );
  }
  return 0;
}

// packages/agent-cli/src/renew.ts
var import_node_crypto6 = require("node:crypto");
var import_node_fs10 = require("node:fs");
var sleep2 = (ms) => new Promise((done) => setTimeout(done, ms));
function tryTakeLock(path, owner) {
  try {
    (0, import_node_fs10.writeFileSync)(path, owner, { flag: "wx", mode: 384 });
    return true;
  } catch {
    return false;
  }
}
function clearStaleLock(path) {
  try {
    if (Date.now() - (0, import_node_fs10.statSync)(path).mtimeMs > LOCK_STALE_MS) {
      (0, import_node_fs10.rmSync)(path, { force: true });
    }
  } catch {
  }
}
async function takeLock(home, signal) {
  ensurePrivateDir(home);
  const path = authLockPath(home);
  const owner = `${process.pid}-${(0, import_node_crypto6.randomBytes)(8).toString("hex")}`;
  while (!signal.aborted) {
    if (tryTakeLock(path, owner)) {
      return owner;
    }
    clearStaleLock(path);
    await sleep2(LOCK_POLL_MS);
  }
  return null;
}
function releaseLock(home, owner) {
  const path = authLockPath(home);
  try {
    if ((0, import_node_fs10.readFileSync)(path, "utf8") === owner) {
      (0, import_node_fs10.rmSync)(path, { force: true });
    }
  } catch {
  }
}
async function renewSignIn(home, now, rejected, signal) {
  const owner = await takeLock(home, signal).catch(() => null);
  if (owner === null) {
    return;
  }
  try {
    const auth = readAuth(home);
    if (!auth || isSignedOut(auth)) {
      return;
    }
    const due = auth.expiresAt - now <= RENEW_MARGIN_MS;
    const refused = rejected !== null && fingerprint(auth.accessToken) === rejected;
    if (!due && !refused) {
      return;
    }
    const result = await refreshPair(auth.tokenEndpoint, auth.clientId, auth.refreshToken, signal);
    if (result.ok) {
      writeAuth(home, {
        ...auth,
        accessToken: result.pair.accessToken,
        refreshToken: result.pair.refreshToken,
        expiresAt: now + result.pair.expiresIn * 1e3
      });
      return;
    }
    const latest = readAuth(home);
    const stillOurs = latest !== null && !isSignedOut(latest) && latest.refreshToken === auth.refreshToken;
    if (result.kind === "invalid_grant" && stillOurs) {
      writeAuth(home, { version: 1, baseUrl: auth.baseUrl, signedOut: true });
    }
  } catch {
  } finally {
    releaseLock(home, owner);
  }
}

// packages/agent-cli/src/version.ts
var CLI_VERSION = true ? "1.0.2" : "development";

// packages/agent-cli/src/main.ts
var USAGE = [
  "Usage: timetrakkr <command>",
  "",
  "  login [--url <address>] [--no-browser] [--detach]",
  "                      Sign in through your browser, picking the workspace there.",
  "                      --url is only for staging or a custom domain, and is",
  "                      remembered. --no-browser only prints the link, to open on",
  "                      this machine yourself. --detach returns once the link is",
  "                      known and waits for the browser in the background, for use",
  "                      from inside an agent session. On a machine with no browser",
  "                      at all, set TIMETRAKKR_TOKEN instead.",
  "  logout              Revoke the sign-in and delete it from this machine. Sessions",
  "                      stop opening the browser to sign in until you sign in again.",
  "  status              What this directory is tracked to, and whether sign-in works",
  "  off                 Stop tracking the current agent session",
  "  on                  Resume tracking the current agent session",
  '  estimate --hours <n> --label "<text>"',
  "                      Record how long the agent said a piece of work would take a",
  "                      person or a team. Sent with the session for comparison; never",
  "                      billed. --days and --weeks also work and can be combined:",
  `                      a day is ${HOURS_PER_DAY} hours and a week is ${DAYS_PER_WEEK} days.`,
  "  hook <host> <event> Called by an agent host; reads the hook payload on stdin",
  "  --version           Print the version"
].join("\n");
async function runHook(rt, host, event) {
  if (!isHost(host) || !event) {
    return;
  }
  let timer;
  const budget = new Promise((done) => {
    timer = setTimeout(done, HOOK_BUDGET_MS);
  });
  const work = handleHook(rt, host, event).catch((error) => {
    logHookError(stateHome(rt.env), rt.now(), `hook ${host} ${event}`, error);
  });
  await Promise.race([work, budget]);
  clearTimeout(timer);
}
async function runRefresh(rt, args) {
  const rejected = args[0] === "--rejected" && args[1] ? args[1] : null;
  const signal = AbortSignal.timeout(RENEW_TIMEOUT_MS);
  await renewSignIn(stateHome(rt.env), rt.now(), rejected, signal);
}
async function run(argv, rt) {
  const [command, ...rest] = argv;
  if (command === "hook") {
    await runHook(rt, rest[0], rest[1]);
    return 0;
  }
  if (command === "refresh") {
    await runRefresh(rt, rest);
    return 0;
  }
  try {
    switch (command) {
      case "login":
        return await login(rt, rest);
      case "logout":
        return await logout(rt);
      case "status":
        return await printStatus(rt);
      case "off":
        return switchTracking(rt, true);
      case "on":
        return switchTracking(rt, false);
      case "estimate":
        return recordEstimate(rt, rest);
      case "version":
      case "--version":
      case "-v":
        rt.out(CLI_VERSION);
        return 0;
      case "help":
      case "--help":
      case "-h":
        rt.out(USAGE);
        return 0;
      default:
        rt.err(USAGE);
        return 1;
    }
  } catch (error) {
    rt.err(`timetrakkr: ${error instanceof Error ? error.message : "unexpected error"}`);
    return 1;
  }
}

// packages/agent-cli/src/cli.ts
function readStdin() {
  return new Promise((done) => {
    let data = "";
    const timer = setTimeout(() => done(data), STDIN_BUDGET_MS);
    const finish = () => {
      clearTimeout(timer);
      done(data);
    };
    process.stdin.setEncoding("utf8");
    process.stdin.on("data", (chunk) => {
      data += chunk;
    });
    process.stdin.on("end", finish);
    process.stdin.on("error", finish);
  });
}
function detached(command, args) {
  try {
    const child = (0, import_node_child_process2.spawn)(command, [...args], { detached: true, stdio: "ignore" });
    child.on("error", () => void 0);
    child.unref();
  } catch {
  }
}
function openBrowser(url) {
  const opener = browserCommand(process.platform, url);
  if (opener) {
    detached(opener.command, opener.args);
  }
}
var runtime = {
  env: process.env,
  cwd: process.cwd(),
  now: Date.now,
  stdin: readStdin,
  out: (text) => process.stdout.write(`${text}
`),
  err: (text) => process.stderr.write(`${text}
`),
  background: (args) => {
    const script = process.argv[1];
    if (script) {
      detached(process.execPath, [script, ...args]);
    }
  },
  openBrowser
};
var isHook = process.argv[2] === "hook";
function exit(code) {
  process.stdout.write("", () => process.exit(code));
}
run(process.argv.slice(2), runtime).then(exit, () => exit(isHook ? 0 : 1));
