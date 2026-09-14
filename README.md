# Windows Update Diagnostic Notes

![Windows](https://img.shields.io/badge/Windows-10%20diagnostics-blue)

## Problema investigado

Documentação técnica de diagnóstico de um problema de atualização do Windows 10 onde foram analisados componentes do sistema, recursos opcionais, serviços e enumeração de dispositivos.

## Ambiente

- Windows 10 22H2
- Build da imagem: `10.0.19045.7725`
- DISM: `10.0.19041.3636`
- PowerShell
- Notebook HP
- Intel Core i5-7200U

## Diagnóstico executado

### Integridade do Component Store

```powershell
DISM /Online /Cleanup-Image /ScanHealth
```

Resultado:

```
Nenhuma corrupção de repositório de componentes detectada.
```

Conclusão: não havia corrupção no armazenamento de componentes do Windows.

---

### Limpeza de componentes antigos

```powershell
DISM /Online /Cleanup-Image /StartComponentCleanup
```

Resultado:

```
A operação foi concluída com êxito.
```

---

## Recurso Print/Fax/Scan

Verificação:

```powershell
Get-WindowsCapability -Online | Where-Object {$_.Name -match "Print.Fax.Scan"}
```

Resultado:

```
Print.Fax.Scan~~~~0.0.1.0
State : Installed
```

Conclusão: o recurso estava instalado.

---

## Windows Image Acquisition (WIA)

O serviço correto no Windows é `stisvc`.

Verificação:

```powershell
Get-Service stisvc
```

Inicialização:

```powershell
Start-Service stisvc
```

Resultado:

```
Status Running
```

---

## Dispositivos de imagem

Comandos usados:

```powershell
pnputil /scan-devices
pnputil /enum-devices /class Image
```

Resultado:

```
Nenhum dispositivo foi encontrado no sistema.
```

Interpretação:

Não havia scanner ou multifuncional conectada. Isso não indicava falha do Windows Update.

---

## Estado PnP

Comando:

```powershell
pnputil /enum-devices /connected
```

Dispositivos relevantes:

- HP TrueVision HD Camera
- Controlador USB 3.0 Intel
- Dispositivos Realtek
- Componentes Intel carregados corretamente

---

# Conclusão técnica

Resultados encontrados:

✅ Component Store íntegro

✅ DISM concluído com sucesso

✅ Recurso Print/Fax/Scan instalado

✅ Serviço WIA funcional

✅ Nenhum scanner físico detectado

✅ Nenhum problema de impressora relacionado ao caso

O problema original de atualização não estava relacionado a scanner, impressora ou corrupção do Windows.

A investigação deve continuar em:

- Windows Update Agent
- Cache de atualização
- Serviços `wuauserv`, `bits` e `cryptsvc`
- Logs CBS/DISM/Windows Update

## Próximos comandos

```powershell
Get-WindowsUpdateLog

net stop wuauserv
net stop bits
net stop cryptsvc
```

Logs importantes:

```
C:\Windows\Logs\CBS\CBS.log
C:\Windows\Logs\DISM\dism.log
```

## Tags

`windows-10` `windows-update` `dism` `powershell` `troubleshooting` `sysadmin` `microsoft`

## Histórico

Documento criado a partir de uma investigação prática de diagnóstico em Windows 10.
