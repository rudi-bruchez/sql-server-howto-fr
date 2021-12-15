# Diagnostic d'AlwaysOn

1. Récupérez les journaux de SQL Server que vous trouverez dans le répertoires de log de SQL Server
1. Récupérez les fichiers de la session d'évènements AlwaysOn_health : fichiers `alwayson_health*.xel` dans le répertoire de log de SQL Server.

Vous trouvez le répertoire de log de SQL Server à l'aide de la requête SQL suivante :

```sql
SELECT 
	LEFT(CAST(SERVERPROPERTY('ErrorLogFileName') as nvarchar(max)), 
		LEN(CAST(SERVERPROPERTY('ErrorLogFileName') as nvarchar(max))) - LEN('ERRORLOG'));
``` 

## Diagnostic de WSFC

Vous pouvez exporter les journaux du cluster à l'aide de la cmdlet `Get-ClusterLog`

```powershell
Import-Module FailoverClusters
Get-ClusterLog -UseLocalTime -Destination C:\temp\
```

Microsoft a créé un outil de récupération du diagnostic du cluster et des journaux. Vous pouvez l'installer en suivant [les instructions suivantes](https://docs.microsoft.com/fr-fr/azure-stack/hci/manage/collect-diagnostic-data#installing-get-sddcdiagnosticinfo-with-powershell).

```powershell
Install-PackageProvider NuGet -Force
Install-Module PrivateCloud.DiagnosticInfo -Force
Import-Module PrivateCloud.DiagnosticInfo -Force
Install-Module -Name MSFT.Network.Diag
```
Ensuite, pour appeler la cmdlet et générer le rapport dans un répertoire.

```powershell
New-Item -Name clusterdiag -Path c:\temp -ItemType Directory -Force
Get-SddcDiagnosticInfo -ClusterName cluster -WriteToPath c:\temp\clusterdiag
```

<p align="right">
<i><small>[<a href="https://www.pachadata.com/contact/">Besoin de services avec SQL Server ? Contactez-moi</a>]</small></i>
</p>