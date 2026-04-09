Если все галочки при установке нажали, то exe лежит в bin папке версии

![[Pasted image 20260409155842.png|590]]

Два файла:
	 rac - утилита 
	 ras - сервис

Если есть, хорошо, если нет, надо установить 

Запустить ras как службу, можно один раз через консоль 
```bash
sc create "1C RAS" binPath= "\"C:\Program Files\1cv8\8.3.27.1859\bin\ras.exe\" --service --port=1545" start= auto
```
В хэлпе ключи есть

Но смысла от этого мало

Проще в автозапуск

```bash
@echo off
set SrvUser=.\USR1CV8 ##От кого запущено
set SrvPwd=123Qwer ##Пароль
set AgentName=localhost # кого слушает
set CtrlPort=1540 # на каком порту
set RASPort=1545 # на каком порту работает служба
set SrvDesc=1C:Enterprise 8.3 Remote Administration Server
set SrvName=1C:Enterprise 8.3 RAS
set SrvName=%SrvName% %AgentName%:%CtrlPort%
set SrvBin=C:\Program Files\1cv8\current\bin\ras.exe
set SrvBin=%SrvBin% cluster --service
set SrvBin=%SrvBin% --port=%RASPort% %AgentName%:%CtrlPort%
tasklist /svc /fi "imagename eq ras.exe" ^
   /fo LIST | find "%SrvName%" > NUL
if not ERRORLEVEL 1 (
  echo Stopping service "%SrvName%"...
  net stop "%SrvName%" 1> NUL
)
sc query "%SrvName%" | find "%SrvName%" > NUL
if not ERRORLEVEL 1 (
  sc delete "%SrvName%" 1> NUL
  echo Deleting service "%SrvName%"...
  ping 127.0.0.1 -n 6 > nul
)
sc create "%SrvName%" binPath= "%SrvBin%" start= auto ^
   obj= "%SrvUser%" password= "%SrvPwd%"
sc description "%SrvName%" "%SrvDesc%"
net start "%SrvName%"
```

Запустить просто батник, само всё сделается, появится служба

Или мышкой

https://infostart.ru/1c/articles/810752/ Статья с инструкцией 
https://its.1c.ru/db/freshex2#content:642:hdoc ещё одна статья (отсюда батник)

Смотрим что заработало

![[Pasted image 20260409161109.png]]

Пользуемся rac там есть help но суть вкратце

Смотрим какие сервера у нас есть


```bash 
rac cluster list
```
Получим id кого хотим смотреть

Например
![[Pasted image 20260409161811.png]]

Его и мониторим, например: 

$cluster наш id кластера к которому подключаемся 

Лицензии проще смотреть в сессиях, например:

```powershell
rac session --cluster=$cluster list --licenses
```
![[Pasted image 20260409162152.png]]

Для примера количество занятых лицензий:

```powershell
(rac session --cluster=$cluster list --licenses | Select-String "license-type").Count
```
![[Pasted image 20260409162345.png]]

Даст цифру, ну парсить как угодно и что угодно, по сути это та же консоль, только для мониторинга удобнее
![[Pasted image 20260409162527.png]]
И пусть себе собирается статистика, всё что есть в оснастке всё тоже самое и здесь
![[Pasted image 20260409162618.png]]

![[Pasted image 20260409162644.png]]

Тоже самое, но в консоли