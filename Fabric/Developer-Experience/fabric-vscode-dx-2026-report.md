# Microsoft Fabric Developer Experience in VS Code (May 2026)

> **Key finding:** Fabric now has a coherent VS Code story built around four Microsoft-published extensions (Microsoft Fabric, Fabric Data Engineering, Fabric MCP, Fabric Skills for Copilot CLI) plus a GA TMDL extension, the GA `fab` CLI (`ms-fabric-cli`), and the GA `fabric-cicd` Python library. Workspace Git integration covers the major item types, and PBIR is rolling out as the default Power BI report format.
> **Confidence:** HIGH for tooling state and install steps; MODERATE for PBIP GA timing (Microsoft says "planned for 2026" but not yet GA at this writing) and for which items are GA vs. preview in Git integration (the Learn list explicitly mixes both).

---

## 1. Fabric VS Code extensions (current and recommended)

As of May 2026 there are **four Microsoft-published Fabric extensions** plus the **TMDL** language extension that together form the recommended stack ([Microsoft Fabric VS Code extensions, Learn](https://learn.microsoft.com/en-us/fabric/data-engineering/set-up-fabric-vs-code-extension)):

| Extension | Marketplace ID | Role | Auth |
|---|---|---|---|
| **Microsoft Fabric** | `fabric.vscode-fabric` | Core: workspace explorer, create/delete/rename items, clone Git-enabled workspaces, Fabric MCP server for Copilot Chat | `Fabric: Sign in` -> browser; uses VS Code Accounts ([Marketplace](https://marketplace.visualstudio.com/items?itemName=fabric.vscode-fabric)) |
| **Fabric Data Engineering** (formerly Synapse VS Code) | `SynapseVSCode.synapse` | Notebooks (Local + VFS modes), Spark Job Definitions, Lakehouse explorer, Environments | `Fabric Data Engineering: Sign In` -> browser ([Get started, Learn](https://learn.microsoft.com/en-us/fabric/data-engineering/setup-vs-code-extension)) |
| **Fabric MCP** | `fabric.fabric-mcp` | Model Context Protocol server enabling AI agents and tools (e.g., Copilot Chat, Claude, custom agents) to interact with Fabric workspaces programmatically | Inherits from Microsoft Fabric extension ([Marketplace](https://marketplace.visualstudio.com/items?itemName=fabric.fabric-mcp)) |
| **Fabric Skills for Copilot CLI** | `fabric.copilot-cli-skills` | Adds Fabric-aware skills to GitHub Copilot CLI for workspace management, notebook authoring, deployment, and diagnostics directly from the terminal | Inherits from Microsoft Fabric extension ([Marketplace](https://marketplace.visualstudio.com/items?itemName=fabric.copilot-cli-skills)) |
| **TMDL** (language support) | `analysis-services.TMDL` | Semantic highlighting, autocomplete, diagnostics, formatting for `.tmdl` files; GA Nov 2025 ([Marketplace](https://marketplace.visualstudio.com/items?itemName=analysis-services.TMDL); [GitHub](https://github.com/microsoft/vscode-tmdl)) |

**Install:** Extensions view -> search by name -> Install. Prerequisites: VS Code, the Jupyter extension (for Data Engineering).

**Auth model:** All Fabric extensions use interactive Microsoft Entra (browser) sign-in via VS Code Accounts; credentials are cached in the OS secure storage. Tenant switching is built into the status bar.

**Companion CLI tools:**

| Tool | Install | Role | Auth |
|---|---|---|---|
| **GitHub Copilot CLI** | `gh extension install github/gh-copilot` (requires `gh` CLI) | AI-powered terminal assistant for code generation, shell commands, and Git workflows; pairs with Fabric Skills extension for Fabric-aware assistance | GitHub auth via `gh auth login` ([GitHub Copilot in the CLI docs](https://docs.github.com/en/copilot/github-copilot-in-the-cli)) |
| **Fabric CLI (`fab`)** | `pip install ms-fabric-cli` (Python 3.10-3.13) | File-system-style wrapper over Fabric REST/OneLake APIs for workspace provisioning, item CRUD, job execution, and scripted CI/CD | `fab auth login` -> browser; supports SPN (`-u`/`-p`) and managed identity (`--identity`) ([GitHub](https://github.com/microsoft/fabric-cli); [PyPI](https://pypi.org/project/ms-fabric-cli/)) |

## 2. Notebook sync workflow

The Fabric Data Engineering extension is the official notebook IDE. It offers two modes ([Get started, Learn](https://learn.microsoft.com/en-us/fabric/data-engineering/setup-vs-code-extension); [Author notebooks in VS Code, Learn](https://learn.microsoft.com/en-us/fabric/data-engineering/author-notebook-with-vs-code)):

- **Local mode** -- pick the workspace, download a notebook to a local folder (`Fabric Data Engineering: Set Local Work Folder`), edit, then sync back. Conflicts are surfaced explicitly.
- **VFS mode** -- "Open Fabric Data Engineering Workspaces" via the Remote Window button to edit items directly without download; supports multiple workspaces in one window.

**Run targets:** Cells can run on the **remote Fabric Spark compute** (full notebookutils available) or on a local Python kernel. The Fabric portal exposes an **Open in VS Code** button that hands the notebook off to the extension.

**Known limitations** ([Notebook limitations, Learn](https://learn.microsoft.com/en-us/fabric/data-engineering/notebook-limitation)):
- Notebook content and snapshot capped at 32 MB; max 256 cells; rich table output 10K rows / 5 MB.
- `mssparkutils` is the legacy alias; current name is **notebookutils**. Magics like `%run` work on Fabric Spark; depth limited to 5.
- Local execution does not provide notebookutils -- a community `notebookutils-interface` shim exists for local stubbing but is not Microsoft-supported.
- 7-day max job run; 60-day job history retention.

## 3. Workspace items authorable from VS Code

| Item | Authoring path |
|---|---|
| Workspace, folders | Microsoft Fabric extension (create/rename/delete items at workspace level; subfolder item creation not yet supported) |
| Notebook | Fabric Data Engineering (CRUD, run on remote Spark) |
| Spark Job Definition | Fabric Data Engineering |
| Lakehouse | Fabric Data Engineering (browse Tables/Files, copy paths); CRUD via core extension or `fab` CLI |
| Environment | Fabric Data Engineering (inspect libraries, Spark config) |
| User Data Function | Fabric User data functions extension |
| SQL Database / Warehouse | Microsoft Fabric extension hands off to **MSSQL** extension ([Connect Lakehouse to VS Code, TW-Waytek](https://www.tw-waytek.com/connect-a-fabric-lakehouse-to-vs-code-in-minutes/)) |
| Semantic model (`.tmdl`) | TMDL extension (language server) + Power BI Desktop apply / fabric-cicd publish |
| Report (`.pbir`) | Power BI Desktop authoring + VS Code for diff/edit; deploy via Git integration or fabric-cicd |
| Data Pipelines | Authored in portal; deployed from Git/fabric-cicd |

## 4. Reports / Power BI

**PBIR (Power BI Enhanced Report Format)** is the new folder/JSON-based report format that replaces the binary report layer inside `.pbix` and `.pbip` ([PBIR will become default, Power BI blog, Nov 2025 / Mar 2026 update](https://powerbi.microsoft.com/en-in/blog/pbir-will-become-the-default-power-bi-report-format-get-ready-for-the-transition/)):

- **Power BI Service:** PBIR became the default for new reports starting January 2026; rollout completing through April 2026 (auto-upgrade for reports with <100 visuals).
- **Power BI Desktop:** PBIR-as-default delayed to the May 2026 Desktop release. Until then it is opt-in via Preview Features "Store reports using enhanced metadata format (PBIR)".
- **PBIP** = project container holding the **PBIR** report folder + the **TMDL** semantic model definition. **PBIP GA is "planned for 2026" but not yet announced as GA**, so treat the file format as production-usable but officially preview-class for contractual purposes.

**TMDL editing** is done with the **TMDL VS Code extension** (`analysis-services.TMDL`), which went GA November 2025 with DAX semantic highlighting, Power Query support, code actions and formatting ([TMDL GA announcement, Power Community](https://www.powercommunity.com/tmdl-visual-studio-code-extension-generally-available/)). Power BI Desktop also has a built-in TMDL view ([Learn](https://learn.microsoft.com/en-us/power-bi/transform-model/desktop-tmdl-view)).

**Report sync to Fabric:** Recommended path is a PBIP folder committed to Git, with the workspace **Git-integrated** (so commits flow back into the workspace). For programmatic deployment use **fabric-cicd**, which natively supports Reports and Semantic Models.

## 5. Source control / Git integration

Workspace-level integration with **Azure DevOps and GitHub** ([Overview, Learn](https://learn.microsoft.com/en-us/fabric/cicd/git-integration/intro-to-git-integration); [Get started, Learn](https://learn.microsoft.com/en-us/fabric/cicd/git-integration/git-get-started)):

- **Auth:** Azure DevOps via OAuth2 or service principal. GitHub via classic or fine-grained PAT. (Optional GitHub Enterprise via repo URL.)
- **Supported items today** include Notebooks, Lakehouses (metadata + shortcuts -- GA per [Lakehouse git, Learn](https://learn.microsoft.com/en-us/fabric/data-engineering/lakehouse-git-deployment-pipelines)), Reports, Paginated Reports, Semantic Models, Data Pipelines, Environments, User Data Functions, and others. The Learn page explicitly notes "Some of the items for Git integration are in preview" -- check the supported-items table in the docs for any specific item type before betting CI/CD on it.
- **Workflow:** Workspace settings -> Git integration -> connect to org/project/repo/branch/folder -> Connect and sync. Subsequent commits use the workspace **Source control** pane. **Branch-out** lets a developer spin up a new feature workspace bound to a new branch directly from the workspace UI.

## 6. fabric-cli (`fab`)

**GA, official Microsoft tool** ([microsoft/fabric-cli](https://github.com/microsoft/fabric-cli); [PyPI v1.6.1, Apr 28 2026](https://pypi.org/project/ms-fabric-cli/); [Learn](https://learn.microsoft.com/en-us/rest/api/fabric/articles/fabric-command-line-interface)).

```bash
pip install ms-fabric-cli       # Python 3.10-3.13
fab auth login                  # interactive browser; or -u/-p for SPN; --identity for MI
fab ls                          # list workspaces
fab mkdir "Sales.Workspace/MyNotebook.Notebook"
fab cp ./local.csv "Sales.Workspace/Lake.Lakehouse/Files/local.csv"
fab job run "Sales.Workspace/MyNotebook.Notebook"
fab import "Prod.Workspace/Data.Lakehouse" -i ./artifacts/Data.Lakehouse
fab api workspaces -q "value[?name=='Sales']"
```

It is a thin file-system-style wrapper over the Fabric REST, OneLake, and `Microsoft.Fabric` ARM endpoints, so anything addressable via REST is reachable. It is the recommended tool for **interactive admin tasks and scripted CI/CD steps that mutate the platform** (creating workspaces, assigning capacity, running pipelines).

## 7. CI/CD pattern (2026)

The recommended modern stack is **Git integration as the source of truth, plus the `fabric-cicd` Python library for deployment**, optionally combined with `fab` for workspace provisioning ([Fabric ci-cd, Learn](https://learn.microsoft.com/en-us/rest/api/fabric/articles/fabric-ci-cd); [microsoft/fabric-cicd v1.0.0 Apr 2026](https://github.com/microsoft/fabric-cicd)).

```bash
pip install fabric-cicd
```

```python
from fabric_cicd import FabricWorkspace, publish_all_items, unpublish_all_orphan_items
ws = FabricWorkspace(
    workspace_id="<guid>",
    repository_directory="./fabric",
    item_type_in_scope=["Notebook","DataPipeline","Environment","SemanticModel","Report"],
)
publish_all_items(ws)
unpublish_all_orphan_items(ws)
```

**Patterns** ([Kevin Chant -- GitHub Actions sample](https://www.kevinrchant.com/2025/07/10/operationalize-fabric-workspaces-using-fabric-cli-and-fabric-cicd-with-github-actions/); [microsoft/fabric-toolbox](https://github.com/microsoft/fabric-toolbox)):

1. **Git-based deploy (recommended):** dev/test/prod workspaces, each connected to its branch; PR merges trigger a GitHub Actions or Azure DevOps pipeline that installs both `ms-fabric-cli` (provisioning) and `fabric-cicd` (item deploy) and runs `publish_all_items`.
2. **Fabric Deployment Pipelines:** UI-driven dev/test/prod promotion. Still supported and useful for less code-centric teams; combine with Git for traceability.

`fabric-cicd` v1.0.0 was released April 20, 2026 and supports Notebooks, Data Pipelines, Environments, Semantic Models, Reports (with parameterization for environment-specific values).

## 8. Recommended green-field "day 1" setup

1. **Sign up** for the [Microsoft Fabric Free Trial](https://www.microsoft.com/microsoft-fabric/getting-started).
2. **Install VS Code**, then install: **Microsoft Fabric**, **Fabric Data Engineering**, **TMDL**, **Jupyter**, **MSSQL**. Add **Fabric User data functions** if relevant.
3. From Command Palette run **`Fabric: Sign in`**. The Fabric explorer appears in the activity bar.
4. **Create a workspace** (`+` in the explorer or `fab mkdir Demo.Workspace`), then **create a Lakehouse and a Notebook** using `+` in the workspace tree.
5. **Edit the notebook locally** (Local or VFS mode), run it on Fabric Spark, save back.
6. In the Fabric portal, **Workspace settings -> Git integration**, connect to a GitHub or Azure DevOps repo, then commit from the **Source control** pane.
7. (Optional) `pip install fabric-cicd` and add a GitHub Actions workflow to deploy items to a separate test workspace on PR merge.

---

## Demo flow recommendation (20-25 min, green-field user)

| # | Step | Time | Tools shown |
|---|---|---|---|
| 1 | **Show Fabric trial sign-up + portal home**, then pivot: "everything from here on lives in VS Code." | 2 min | Browser |
| 2 | **Install the three extensions** from Marketplace (Microsoft Fabric, Fabric Data Engineering, TMDL); run `Fabric: Sign in`. Show the Fabric explorer populating. | 3 min | VS Code |
| 3 | **Create a workspace + lakehouse** from VS Code (`+` in explorer). Drop a CSV into Files via drag-drop or `fab cp`. | 3 min | Fabric core ext, optional `fab` CLI |
| 4 | **Create and run a notebook** (Local mode). Type one PySpark cell that reads the CSV, run it on remote Fabric Spark; save -> sync back to workspace. Briefly mention notebookutils and the 32 MB limit. | 5 min | Fabric Data Engineering ext |
| 5 | **Open a PBIP project** containing a `.pbir` report and a `definition/` TMDL semantic model. Edit a measure in TMDL with autocomplete; show diff in VS Code Source Control. | 4 min | TMDL ext, Git pane |
| 6 | **Connect the workspace to GitHub** in the Fabric portal; back in VS Code use **Branch-out** to create a feature workspace + branch; commit the TMDL change; show it appearing in the Fabric workspace after Update. | 4 min | Fabric portal Git integration |
| 7 | **CI/CD teaser:** show a 10-line GitHub Actions YAML that does `pip install ms-fabric-cli fabric-cicd`, `fab auth login` with a service principal, and `publish_all_items` to a target workspace. Trigger it; show the deployment land. | 3 min | GitHub Actions, fabric-cli, fabric-cicd |

Concrete commands to keep in your demo cheat-sheet:
- `Fabric: Sign in` / `Fabric Data Engineering: Sign In`
- `Fabric Data Engineering: Set Local Work Folder`
- `fab auth login` / `fab ls` / `fab mkdir Demo.Workspace` / `fab job run Demo.Workspace/Hello.Notebook`
- `pip install ms-fabric-cli fabric-cicd`

---

## Conclusion

The 2026 Fabric VS Code experience has matured from an experimental Synapse fork into a coherent four-extension stack with first-class Git, a GA CLI (`fab`), and a GA deployment library (`fabric-cicd`). The biggest moving piece is Power BI's PBIR/PBIP transition: PBIR is now the Service default and is on track to be Desktop default in May 2026, but PBIP GA is still pending, so document any production CI/CD pattern as "preview format, GA expected 2026." Otherwise every layer of the demo flow above rests on GA, Microsoft-supported tooling.

### Risks and limitations
- **PBIP GA timing** is a moving target; Microsoft has reiterated "planned for 2026" but no GA date is published as of May 2026 ([Power BI blog](https://powerbi.microsoft.com/en-in/blog/pbir-will-become-the-default-power-bi-report-format-get-ready-for-the-transition/)). For a customer demo, present PBIR/PBIP as the strategic direction without contractual GA promises.
- **Git integration item coverage**: some item types remain preview-tagged in the Learn supported-items list. Validate per item before scoping a customer's CI/CD.
- **Local notebook execution** does not natively support notebookutils; there is no Microsoft-supported local stub yet -- demos should run on remote Fabric Spark.
- **Service principal coverage** in Fabric REST APIs is improving but still incomplete for some CI/CD operations; the [`fabric-toolbox` accelerator](https://github.com/microsoft/fabric-toolbox) documents workarounds.

## Sources

1. [Microsoft Fabric VS Code extensions, Learn](https://learn.microsoft.com/en-us/fabric/data-engineering/set-up-fabric-vs-code-extension)
2. [Get started with the Fabric Data Engineering VS Code extension, Learn](https://learn.microsoft.com/en-us/fabric/data-engineering/setup-vs-code-extension)
3. [Microsoft Fabric extension - Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=fabric.vscode-fabric)
4. [Fabric Data Engineering VS Code - Marketplace](https://marketplace.visualstudio.com/itemdetails?itemName=SynapseVSCode.synapse)
5. [Fabric User data functions - Marketplace](https://marketplace.visualstudio.com/items?itemName=fabric.vscode-fabric-functions)
6. [Create a Fabric User data functions item in VS Code, Learn](https://learn.microsoft.com/en-us/fabric/data-engineering/user-data-functions/create-user-data-functions-vs-code)
7. [Create and manage Fabric notebooks in VS Code, Learn](https://learn.microsoft.com/en-us/fabric/data-engineering/author-notebook-with-vs-code)
8. [Fabric Notebook known limitations, Learn](https://learn.microsoft.com/en-us/fabric/data-engineering/notebook-limitation)
9. [microsoft/fabric-cli, GitHub](https://github.com/microsoft/fabric-cli)
10. [Fabric CLI documentation site](https://microsoft.github.io/fabric-cli/)
11. [ms-fabric-cli on PyPI (v1.6.1, Apr 2026)](https://pypi.org/project/ms-fabric-cli/)
12. [Fabric command line interface, Learn REST API](https://learn.microsoft.com/en-us/rest/api/fabric/articles/fabric-command-line-interface)
13. [microsoft/fabric-cicd, GitHub](https://github.com/microsoft/fabric-cicd)
14. [fabric-cicd on PyPI (v1.0.0, Apr 2026)](https://pypi.org/project/fabric-cicd/)
15. [Fabric ci-cd Python library, Learn](https://learn.microsoft.com/en-us/rest/api/fabric/articles/fabric-ci-cd)
16. [PBIR will become the default Power BI Report Format, Power BI blog](https://powerbi.microsoft.com/en-in/blog/pbir-will-become-the-default-power-bi-report-format-get-ready-for-the-transition/)
17. [TMDL Extension - Marketplace](https://marketplace.visualstudio.com/items?itemName=analysis-services.TMDL)
18. [microsoft/vscode-tmdl, GitHub](https://github.com/microsoft/vscode-tmdl)
19. [TMDL view in Power BI Desktop, Learn](https://learn.microsoft.com/en-us/power-bi/transform-model/desktop-tmdl-view)
20. [Overview of Fabric Git integration, Learn](https://learn.microsoft.com/en-us/fabric/cicd/git-integration/intro-to-git-integration)
21. [Get started with Git integration, Learn](https://learn.microsoft.com/en-us/fabric/cicd/git-integration/git-get-started)
22. [Lakehouse git integration and deployment pipelines, Learn](https://learn.microsoft.com/en-us/fabric/data-engineering/lakehouse-git-deployment-pipelines)
23. [microsoft/fabric-toolbox CI/CD accelerators, GitHub](https://github.com/microsoft/fabric-toolbox)
24. [Operationalize Fabric workspaces with fabric-cli and fabric-cicd, Kevin Chant](https://www.kevinrchant.com/2025/07/10/operationalize-fabric-workspaces-using-fabric-cli-and-fabric-cicd-with-github-actions/)
