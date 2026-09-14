# Windows Update Diagnostic Notes

![Windows](https://img.shields.io/badge/Windows-10%20diagnostics-blue)

## Investigação: Windows Update - diagnóstico de componentes

Este repositório documenta uma investigação prática de falha de atualização do Windows, separando causas reais de hipóteses descartadas.

## Ambiente

- Windows 10 22H2
- Build: `10.0.19045.7725`
- DISM: `10.0.19041.3636`
- PowerShell
- Notebook HP
- Intel Core i5-7200U

---

# 1. Verificação de corrupção do Windows

Comando:

```powershell
DISM /Online /Cleanup-Image /ScanHealth
```

Resultado:

```
Nenhuma corrupção de repositório de componentes detectada.
```

Conclusão:

O Component Store estava íntegro.

---

# 2. Limpeza do Component Store

```powershell
DISM /Online /Cleanup-Image /StartComponentCleanup
```

Resultado:

```
A operação foi concluída com êxito.
```

---

# 3. Investigação de Print/Scan/WIA

Hipótese avaliada: problema de impressora ou scanner.

Recurso:

```powershell
Get-WindowsCapability -Online | Where-Object {$_.Name -match "Print.Fax.Scan"}
```

Resultado:

```
Print.Fax.Scan~~~~0.0.1.0
State : Installed
```

Serviço WIA:

```powershell
Get-Service stisvc
Start-Service stisvc
```

Resultado:

```
Status: Running
```

Conclusão:

Scanner, impressora e WIA foram descartados como causa da atualização.

---

# 4. Enumeração de dispositivos

Comandos:

```powershell
pnputil /scan-devices
pnputil /enum-devices /class Image
```

Resultado:

```
Nenhum dispositivo foi encontrado no sistema.
```

Interpretação:

Não havia scanner físico conectado. Isso era esperado e não explicava a falha do Windows Update.

---

# 5. Diagnóstico PnP

Dispositivos encontrados funcionando:

- HP TrueVision HD Camera
- Intel USB 3.0 Controller
- Realtek Wi-Fi/Ethernet
- Componentes Intel

---

# Conclusão

Resultados confirmados:

✅ Windows Component Store íntegro  
✅ DISM executado com sucesso  
✅ Recursos opcionais instalados  
✅ WIA funcional  
✅ Scanner/impressora descartados como causa  

O foco correto da investigação permanece:

- Windows Update Agent
- Cache de atualização
- Serviços `wuauserv`, `bits`, `cryptsvc`
- Logs CBS/DISM

---

# Próximos comandos

```powershell
Get-WindowsUpdateLog

Get-Service wuauserv,bits,cryptsvc,msiserver
```

Logs:

```
C:\Windows\Logs\CBS\CBS.log
C:\Windows\Logs\DISM\dism.log
```

---

## Tags

`windows-update` `windows-10` `dism` `powershell` `troubleshooting` `sysadmin` `microsoft`

## Histórico

Documentação criada a partir de diagnóstico real em ambiente Windows 10.
