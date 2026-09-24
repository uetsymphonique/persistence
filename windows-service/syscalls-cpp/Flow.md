# NtServiceInstaller — Flow

**Entry:** `wmain` → `install` command → `InstallService` (main.cpp / service_installer.cpp)  ·  **Artifact summary:** one auto-start service registry key + config values written under `\Registry\Machine\SYSTEM\CurrentControlSet\Services` via ntdll `Nt*` functions; SCM never contacted

| # | Behavior (`actor action artifact`) | Artifact [class] → consumed by | Tactic / TID — Technique Name | Context (baseline) |
|---|---|---|---|---|
| 1 | installer checks own process token for built-in Administrators group membership (AllocateAndInitializeSid + CheckTokenMembership) | — [no-artifact] → #3 | — | privilege gate before HKLM write; token group query only, no token modification |
| 2 | installer resolves NtOpenKey/NtCreateKey/NtSetValueKey/NtClose function pointers from already-loaded ntdll.dll (GetModuleHandleW + GetProcAddress) | — [no-artifact] → #3 | — | Nt* registry functions absent from import table; GetModuleHandle reuses loaded ntdll, no new module-load event |
| 3 | installer creates service registry subkey `<service-name>` under `\Registry\Machine\SYSTEM\CurrentControlSet\Services` (NtOpenKey parent + NtCreateKey, REG_OPTION_NON_VOLATILE, KEY_ALL_ACCESS) | new service key under `HKLM\SYSTEM\CurrentControlSet\Services` [registry] → #4 | — | raw NT object-manager path, not Win32 registry API; no advapi32 CreateServiceW / OpenSCManager; key not visible to SCM until reboot |
| 4 | installer writes service config values into the service key: `Type=0x10` (own process), `Start=0x2` (auto-start), `ErrorControl=0x1`, `ImagePath=<exe-path arg>`, `DisplayName` (defaults to service name), `ObjectName=LocalSystem`, optional `Description` (NtSetValueKey ×6–7, NtClose) | configured auto-start service values [registry] | — | registry-only install; no SCM service-install event (no System 7045 / Security 4697); SCM reads values at next boot |
