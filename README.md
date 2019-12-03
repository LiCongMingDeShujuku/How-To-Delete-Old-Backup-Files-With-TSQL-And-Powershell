![CLEVER DATA GIT REPO](https://raw.githubusercontent.com/LiCongMingDeShujuku/git-resources/master/0-clever-data-github.png "李聪明的数据库")

# 如何使用TSQL和Powershell删除旧备份文件
#### How To Delete Old Backup Files With TSQL And Powershell
**发布-日期: 2017年6月28日 (评论)**



## Contents

- [中文](#中文)
- [English](#English)
- [SQL Logic](#Logic)
- [Build Info](#Build-Info)
- [Author](#Author)
- [License](#License) 


## 中文
以下是允许你使用TSQL删除旧备份文件的逻辑，其中TSQL构建了Powershell删除脚本。


## English
The following logic allows you to delete old backup files with TSQL where the Powershell delete script is built.

![#](images/如何使用TSQL和Powershell删除旧备份文件.png?raw=true "#")

Here’s how it works…
这是它的工作原理......
First; you supply the backup drive letter.
第一; 提供备份drive letter。
Second; you supply the backup retention in number of days. In this example we are using 30 days.
第二; 提供备份保留的天数。 在这个例子中，我们使用了30天。
The process will then create a backup structure under whatever drive you supplied. In this example we are using H:SQLBACKUPS
这个过程会在你提供的任何驱动器下创建一个备份结构。在这个例子中，我们使用H：SQLBACKUPS

Note: This logic assumes the existing backup path was created by the same process, and all backups will go into the respective path. H:SQLBAKUPS If the backup folder structure already exists; then nothing will happen. The process will simply continue with the rest of the process.
注意：此逻辑假定现有备份路径是由同一进程创建的，并且所有备份都将进入相应的路径。 H：SQLBAKUPS如果备份文件夹结构已存在; 什么都不会发生，该过程将继续进行剩余的过程。
In order for you to execute Powershell scripts from TSQL you’ll need to use the xp_cmdshell extended stored procedure. Since the xp_delete_file sp is not supported in future version of TSQL, and Microsoft is pushing for a wider adoption of Powershell (as they should), then it’s best to incorporate Powershell into more automated processes, and until there is another official procedure released for this purpose xp_cmdshell is the recommended approach.
为了从TSQL执行Powershell脚本，你需要使用xp_cmdshell扩展存储过程。由于未来版本的TSQL不支持xp_delete_file sp，并且Microsoft正在推动更广泛地采用Powershell（正如他们应该的那样），因此最好将Powershell合并到更自动化的流程中，直到为此目的发布另一个正式程序，xp_cmdshell是推荐的方法。
Next; the logic will check to see if xp_cmdshell is enabled.
下一步，逻辑将检查是否启用了xp_cmdshell。
If not enabled; it will enable it.
如果未启用; 它会启用它。
Then it will proceed to delete old backup files (older than the retention period).
The final step is it will disable xp_cmdshell.
然后它将继续删除旧的备份文件（早于保留期）。
最后一步是它将禁用xp_cmdshell。


---
## Logic
```SQL
-- this deletes old backup files based on a retention period in days.      
    根据保留期限（以天为单位）删除旧备份文件。
-- make sure to supply the appropriate backup drive and retention period.
    确保提供适当的备份驱动器和保留期
 
use master;
set nocount on
declare @drive          varchar(1)  = 'H'   -- <-- change drive letter to your backup drive
declare @retention      varchar(2)  = '30'  -- <-- change retention to number of days you want to keep
 
-- sets backup location info
declare @server_name        varchar(255)
declare @backup_path        varchar(255)
declare @removeoldfiles     varchar(510)
declare @config         int
set @server_name        = (select replace(cast(serverproperty('servername') as varchar(255)), '\', '--'))
set @backup_path        = (@drive + ':\SQLBACKUPS\' + @server_name)
set @config         = (select cast([value_in_use] as int) from sys.configurations where [configuration_id] = '16390')
set @removeoldfiles     = 'powershell.exe Get-ChildItem "' + @backup_path + '" -Recurse | Where {$_.creationtime -lt (Get-Date).AddDays(-' + @retention + ')} | Remove-Item -Force'
exec    master..xp_create_subdir @backup_path
        if  @config = 0
            begin 
                exec master..sp_configure 'show advanced options', 1 reconfigure 
                exec master..sp_configure 'xp_cmdshell', 1 reconfigure with override
            end
exec    (@removeoldfiles)
waitfor delay '00:00:05'
exec    master..sp_configure 'xp_cmdshell', 0 reconfigure with override


```

[![WorksEveryTime](https://forthebadge.com/images/badges/60-percent-of-the-time-works-every-time.svg)](https://shitday.de/)

## Build-Info

| Build Quality | Build History |
|--|--|
|<table><tr><td>[![Build-Status](https://ci.appveyor.com/api/projects/status/pjxh5g91jpbh7t84?svg?style=flat-square)](#)</td></tr><tr><td>[![Coverage](https://coveralls.io/repos/github/tygerbytes/ResourceFitness/badge.svg?style=flat-square)](#)</td></tr><tr><td>[![Nuget](https://img.shields.io/nuget/v/TW.Resfit.Core.svg?style=flat-square)](#)</td></tr></table>|<table><tr><td>[![Build history](https://buildstats.info/appveyor/chart/tygerbytes/resourcefitness)](#)</td></tr></table>|

## Author

- **李聪明的数据库 Lee's Clever Data**
- **Mike的数据库宝典 Mikes Database Collection**
- **李聪明的数据库** "Lee Songming"

[![Gist](https://img.shields.io/badge/Gist-李聪明的数据库-<COLOR>.svg)](https://gist.github.com/congmingshuju)
[![Twitter](https://img.shields.io/badge/Twitter-mike的数据库宝典-<COLOR>.svg)](https://twitter.com/mikesdatawork?lang=en)
[![Wordpress](https://img.shields.io/badge/Wordpress-mike的数据库宝典-<COLOR>.svg)](https://mikesdatawork.wordpress.com/)

---
## License
[![LicenseCCSA](https://img.shields.io/badge/License-CreativeCommonsSA-<COLOR>.svg)](https://creativecommons.org/share-your-work/licensing-types-examples/)

![Lee Songming](https://raw.githubusercontent.com/LiCongMingDeShujuku/git-resources/master/1-clever-data-github.png "李聪明的数据库")

