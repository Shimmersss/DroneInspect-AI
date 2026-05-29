# Repository Guidelines

## Project Layout

This repository is a Windows-oriented local demo stack for DroneInspect-AI.

- `frontend/`: Vue 3 + Vite client. Vite proxies `/api` to the Spring backend and `/mmdet3d` to the MMDet3D service.
- `backend/`: Spring Boot 3 backend. It owns auth, image upload, YOLO/MMDet3D orchestration, and Dify/agent integration.
- `services/yolo-http/`: FastAPI + Ultralytics YOLO image detection service.
- `services/mmdet3d/`: FastAPI PointPillars/MMDetection3D point-cloud service.
- `services/dify/`: Dify workflow export/configuration files only. It is not a local service launched by `start-all.ps1`.
- `.run/`: generated process metadata and logs. Treat it as runtime output.

Older notes may mention `SOFT-rear/` and `web/`; in this checkout those roles are `backend/` and `frontend/`.

## Local Runtime

Use PowerShell from the repository root.

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File .\start-all.ps1
powershell -NoProfile -ExecutionPolicy Bypass -File .\status-all.ps1
powershell -NoProfile -ExecutionPolicy Bypass -File .\stop-all.ps1
```

Default local ports:

- Frontend: `http://127.0.0.1:3000`
- Backend: `http://127.0.0.1:8080`
- YOLO: `http://127.0.0.1:9000`
- MMDet3D: `http://127.0.0.1:8000`

`start-all.ps1` loads `.env.local` into the process environment before launching services. Keep real secrets in `.env.local`; keep `.env.local.example` tracked.

## Build And Check Commands

Prefer targeted checks before broad runs.

```powershell
cd .\frontend
npm run build
```

```powershell
cd .\backend
mvn -DskipTests package
```

```powershell
cd .\services\yolo-http
python -m py_compile .\app.py
```

```powershell
cd .\services\mmdet3d
python -m py_compile .\app\main.py .\app\detector.py .\app\settings.py .\app\schemas.py
```

Use `mvn.cmd` and `C:\Program Files\nodejs\npm.cmd` directly if wrapper or PowerShell shim behavior is flaky on Windows.

## Configuration Rules

- Do not commit real database passwords, Dify API keys, or machine-specific paths.
- Backend local overrides live in `backend/src/main/resources/application-local.properties` and should read environment variables where possible.
- Root `.env.local` is ignored; `.env.local.example` is intentionally unignored.
- YOLO model resolution is `MODEL_PATH`, then local `services/yolo-http/best.pt` or `services/yolo-http/yolo26m.pt`.
- MMDet3D should prefer repository-local assets when present, but this workspace may still use `KITTI_DATASET_ROOT=F:/YOLO/kitty`.

## Integration Notes

- The frontend axios wrapper returns `response.data` for successful backend responses. Components should generally read payloads from `response?.data`, not `response?.data?.data`.
- The provider-neutral backend abstraction is `AgentDecisionService`.
- Compatibility endpoints for Dify are kept alongside agent endpoints: `/external/dify/status`, `/external/dify/chat`, `/external/agent/status`, and `/external/agent/chat`.
- The backend probes Dify through `DifyWorkflowService`; detection should remain fail-open when the agent/Dify workflow is unavailable.
- Dify service calls are backend-side calls to the Dify service API, not frontend-direct requests and not OpenAI-compatible endpoints.

## Editing Conventions

- Keep changes scoped to the affected module. Avoid repo-wide refactors unless required for the requested behavior.
- Preserve the existing Java package structure under `org.soft.softrear`.
- For frontend work, follow the existing Vue single-file component style and use the current `frontend/src/utils/axios.js` response contract.
- For Python services, keep FastAPI response fields stable because the Java backend and Vue UI depend on them.
- Generated logs, model weights, datasets, virtual environments, and third-party runtime outputs should stay untracked unless the user explicitly asks to package them.

## Verification Preference

When changing integration behavior, verify the layer that owns the visible output:

- Frontend display issues: inspect `frontend/src/utils/axios.js` and the consuming component first.
- YOLO label issues: inspect `services/yolo-http/app.py` response fields.
- MMDet3D rendered image issues: check whether Java redraws or replaces Python-rendered output.
- Dify readiness issues: inspect backend probe/config code before updating documentation.
