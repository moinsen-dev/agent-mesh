# Onboarding: add a new machine as a mesh agent

1. **Prerequisites** (see README): git SSH key on GitHub, `age`, `sops`, `hermes`.
2. **Repo access**: the agent needs push access to the private
   `moinsen-dev/agent-mesh-memories` repo (invite its SSH key as a collaborator).
3. **Install the framework**:
   ```bash
   curl -fsSL https://raw.githubusercontent.com/moinsen-dev/agent-mesh/main/agent-mesh -o /usr/local/bin/agent-mesh
   chmod +x /usr/local/bin/agent-mesh
   ```
4. **Initialize** (unique agent name, e.g. hostname):
   ```bash
   agent-mesh init <agent-name>
   ```
   Creates: age key pair, `~/.agent-mesh/agent-mesh.conf`, clones both repos.
   If `gh` is authenticated, missing repos are created automatically
   (`gh repo create`) — no manual GitHub setup needed.

5. **Push the first registration**:
   ```bash
   agent-mesh sync
   ```
   → creates `agents/<name>/` + `vault/keys/<name>.age.pub`.

## Configuration

The agent reads configuration from three sources (highest priority first):

1. **Environment variables** — `AGENT_MESH_HOME`, `AGENT_MESH_GH_ORG`,
   `AGENT_MESH_PUBLIC_REPO`, `AGENT_MESH_PRIVATE_REPO`, `PYTHON_BIN`
2. **`~/.agent-mesh/agent-mesh.conf`** — written by `agent-mesh init`;
   persists `GH_ORG`, `PUBLIC_REPO`, `PRIVATE_REPO`, `PYTHON_BIN` when
   they differ from defaults
3. **Defaults** — `moinsen-dev`, `agent-mesh`, `agent-mesh-memories`

Example: use your own GitHub account instead of the upstream org:

```bash
export AGENT_MESH_GH_ORG="my-org"
agent-mesh init my-agent   # GH_ORG is persisted to agent-mesh.conf
# later commands work without the env var:
agent-mesh sync
```
6. **Test vault access**:
   ```bash
   agent-mesh vault list    # should show the shared secrets
   agent-mesh vault get <some-secret>
   ```
   (If the vault was encrypted before this agent joined, an existing agent
   must re-set one secret once — then the new key becomes a recipient.)
7. **Set up the cron** (fallback; webhook replaces it on the hub):
   ```bash
   echo "0 6 * * * root /usr/local/bin/agent-mesh sync >> /var/log/mesh-sync.log 2>&1" \
     > /etc/cron.d/mesh-sync
   ```
8. **Assign a role** (default is `worker`):
   ```bash
   agent-mesh role hub        # only ONE hub per mesh (central point of contact)
   agent-mesh role specialist # domain expert, e.g. media, db, web
   ```

## Rules for agents

- **Never put secrets in memories/insights** — vault only.
- **Skills**: `agent-mesh sync` exports agent-created skills only (from
  `~/.hermes/skills/.usage.json`, `created_by: agent`).
- **Conflicts**: `agent-mesh sync` does `git pull --rebase` — resolve conflicts
  manually if two agents changed the same file simultaneously.
- **Mesh health**: `agent-mesh status` shows all agents + vault status.
- **Real-time**: the hub's webhook triggers sync instantly on every push;
  other agents can also run `agent-mesh-webhook.py` + tunnel if they want the same.
