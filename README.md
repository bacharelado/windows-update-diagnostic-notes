# Windows Update Diagnostic Notes

![Windows](https://img.shields.io/badge/Windows-10%20diagnostics-blue)

## Problema investigado

Documentação técnica de diagnóstico de um problema de atualização do Windows 10 onde sintomas relacionados a componentes do sistema, recursos opcionais e dispositivos de imagem/scanner foram investigados.

O objetivo deste repositório é registrar o processo de investigação, comandos utilizados e conclusões para referência futura.

## Ambiente analisado

- Windows 10 Pro
- Build da imagem: `10.0.19045.7725`
- DISM: `10.0.19041.3636`
- PowerShell

## Diagnóstico executado

### Verificação de corrupção do sistema

```powershell
DISM /Online /Cleanup-Image /ScanHealth
```

Resultado:

```
Nenhuma corrupção de repositório de componentes detectada.
```

### Limpeza do repositório de componentes

```powershell
DISM /Online /Cleanup-Image /StartComponentCleanup
```

Resultado:

```
A operação foi concluída com êxito.
```

## Recursos de impressão e digitalização

Verificado:

```powershell
Get-WindowsCapability -Online | Where-Object {$_.Name -match "Print.Fax.Scan"}
```

Resultado:

```
Print.Fax.Scan~~~~0.0.1.0
State : Installed
```

Conclusão: o componente nativo de Fax/Scan estava instalado.

## Serviço WIA

Verificação:

```powershell
Get-Service stisvc
```

Resultado:

```
Assistente de aquisição de imagens do Windows (WIA)
```

O serviço foi iniciado manualmente:

```powershell
Start-Service stisvc
```

Resultado:

```
Status Running
```

## Investigação de dispositivos de imagem

Comandos usados:

```powershell
pnputil /scan-devices
pnputil /enum-devices /class Image
```

Resultado:

```
Nenhum dispositivo foi encontrado no sistema.
```

Conclusão: não havia scanner ou dispositivo WIA físico conectado durante o diagnóstico.

## Verificação de integridade do hardware PnP

```powershell
pnputil /enum-devices /connected
```

Dispositivos relevantes encontrados:

- HP TrueVision HD Camera
- Controladores USB funcionando
- Adaptadores de rede funcionando
- Componentes Intel e Realtek carregados

## Conclusão técnica

Durante a análise:

- O armazenamento de componentes do Windows estava íntegro.
- O recurso Print/Fax/Scan estava instalado.
- O serviço WIA estava funcional.
- Não foi identificado scanner ou impressora física conectada.
- O problema não aparentava ser causado por corrupção do Windows ou ausência do recurso de digitalização.

## Próximas investigações possíveis

Caso o problema de atualização persista:

1. Coletar logs do Windows Update:

```powershell
Get-WindowsUpdateLog
```

2. Verificar componentes do Windows Update:

```powershell
net stop wuauserv
net stop bits
net stop cryptsvc
```

3. Analisar:

- `C:\Windows\Logs\CBS\CBS.log`
- `C:\Windows\Logs\DISM\dism.log`
- logs do Windows Update

## Histórico

Registro criado a partir de uma investigação prática de diagnóstico em Windows 10.

Contribuições e correções são bem-vindas.
