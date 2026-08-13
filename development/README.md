# 💻 Development Workflow Guide (VS Code DevContainers)

For feature development on Windows/Linux/macOS without setting up Python, Node, MariaDB, or Bench natively.

---

## 1. Development Prerequisites

1. **Install Docker Desktop** (Windows/macOS) or **Docker CE** (Linux).
2. **Install Visual Studio Code** and the **[Dev Containers Extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)** (`ms-vscode-remote.remote-containers`).

---

## 2. Understanding Development Environment Files

| File Path | Purpose |
| :--- | :--- |
| **`.devcontainer/<stack>/devcontainer.json`** | VS Code configuration pointing to `../../development/compose.dev.yml` and `initializeCommand` |
| **`.devcontainer/<stack>/docker-compose.override.yml`** | Defines stack project name (`name: stack-1`) and host port bindings (`8000:8000`) |
| **`.devcontainer/<stack>/.env`** | Stack-specific active environment configuration |
| **`.devcontainer/<stack>/.env.example`** | Stack environment template reference |
| **`development/compose.shared.yml`** | Shared infrastructure stack (`dev-mariadb`, `dev-redis-cache`, `dev-redis-queue`) |
| **`development/compose.dev.yml`** | Development container Compose stack connected to `frappe-dev-network` |
| **`development/apps.base.json`** | Base ecosystem apps manifest (ERPNext, HRMS, Healthcare, CRM) |

### 💡 Why `apps.base.json` is Used & How to Change It

#### Why it is used:
`development/apps.base.json` specifies the core ecosystem apps (`ERPNext`, `HRMS`, `Healthcare`, `CRM`) pre-installed during container initialization. When VS Code builds your DevContainer, `bench init` reads `apps.base.json` to automatically clone and set up these apps inside `/home/frappe/frappe-bench/apps/`, saving developers from manually fetching standard ecosystem apps every time.

#### How to change or customize it:

1. **Option A: Edit `development/apps.base.json` directly (Applies to all dev stacks)**:
   Open `development/apps.base.json` and add, remove, or update Git URLs and branches:
   ```json
   [
     {
       "url": "https://github.com/frappe/erpnext",
       "branch": "version-16"
     },
     {
       "url": "https://github.com/frappe/hrms",
       "branch": "version-16"
     }
   ]
   ```

2. **Option B: Use a custom base apps file per DevContainer Stack**:
   - Create your manifest file inside `development/` (e.g., `development/apps.team-b.json`).
   - Open `.devcontainer/<stack>/.env` and update `APPS_FILE`:
     ```env
     APPS_FILE=apps.team-b.json
     ```
   - Rebuild the container: Press `F1` $\rightarrow$ select **Dev Containers: Rebuild Container**.

---

## 3. Launching Development Container

By default in VS Code, `Reopen in Container` transforms the active window. To keep your local repository window open and launch the DevContainer in a **NEW window**:

1. Press **`Ctrl+Shift+N`** (or **File $\rightarrow$ New Window**) to open a new VS Code window.
2. In the new window, press **`F1`** (or `Ctrl+Shift+P`) $\rightarrow$ select **Dev Containers: Reopen in Container** (or **Dev Containers: Open Folder in Container...**).
3. Select **Stack 1** from the choice menu.
4. VS Code automatically executes `initializeCommand` to start shared MariaDB & Redis containers (`development/compose.shared.yml`), builds/launches Stack 1 inside the new window, and lands you directly inside `/home/frappe/frappe-bench`.

---

## 4. First-Time Dev Site Creation inside Container

Open the integrated terminal in VS Code inside the container. The Frappe Python virtual environment is **automatically activated** in `$PATH` and `.bashrc` for every terminal tab:

```bash
# 1. Create a new development site connected to the shared MariaDB container
bench new-site --db-host=dev-mariadb --db-root-username=root --db-root-password=admin --admin-password=admin site1.local

# 2. Set default site
bench use site1.local

# 3. Install base applications onto site
bench --site site1.local install-app erpnext hrms healthcare crm

# 4. Start bench development server
bench start
```

Access your application at: `http://localhost:8000`

---

## 5. Installing Custom Apps in Development

Inside the container terminal:

```bash
# 1. Fetch custom app repository
bench get-app <app_name> <git_repository_url> --branch <branch_name>

# 2. Install app onto site
bench --site site1.local install-app <app_name>

# 3. Commit your changes
git status
git commit -m "feat: add custom feature"
```

---

## 6. Creating Additional Stacks (Copy & Paste Workflow)

To run multiple isolated dev stacks simultaneously (e.g. testing different feature branches or projects):

1. **Duplicate Stack Folder**:
   Copy `.devcontainer/stack-1` and paste as `.devcontainer/stack-2` (or `.devcontainer/<my-project-name>`).

2. **Update `docker-compose.override.yml`**:
   Open `.devcontainer/stack-2/docker-compose.override.yml` and update project name, ports, and `.env` path:
   ```yaml
   name: stack-2

   services:
     devcontainer:
       ports:
         - "8005:8000"
         - "9005:9000"
       env_file:
         - ../.devcontainer/stack-2/.env
   ```

3. **Reopen in VS Code**:
   In a new window (`Ctrl+Shift+N`), press `F1` $\rightarrow$ **Dev Containers: Reopen in Container** $\rightarrow$ select **Stack 2**.

> [!NOTE]
> All stacks share the same `dev-mariadb` and `dev-redis` containers over `frappe-dev-network`, saving over **66% RAM** while maintaining 100% database and volume isolation!

---

## 7. Changing Frappe Framework Version in Development

### Option A: In-Container Version Switch (Preserves Database & Volume)
```bash
cd /home/frappe/frappe-bench/apps/frappe
git fetch --tags
git checkout <VERSION_TAG>   # e.g., v16.30.0

cd /home/frappe/frappe-bench
./env/bin/pip install --no-cache-dir -e ./apps/frappe
bench build --app frappe
bench migrate
```

### Option B: Clean Reset (Rebuilds Volume from Scratch)
1. Update `FRAPPE_BRANCH=v16.30.0` in `.devcontainer/stack-1/.env`.
2. In terminal: `docker compose -f development/compose.dev.yml down -v`
3. In VS Code: Press `F1` $\rightarrow$ **Dev Containers: Reopen in Container**.
