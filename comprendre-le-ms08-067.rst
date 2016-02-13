Comprendre le MS08-067
######################
:date: 2016-02-05 15:59
:slug: comprendre-le-ms08-067
:category: Reverse
:status: draft

Introduction :
~~~~~~~~~~~~~~

En 2008, une faille de sécurité allait faire trembler le monde de la
sécurité.Exploitée par l'un des virus les plus célèbres de cette
décennie : https://fr.wikipedia.org/wiki/Conficker,
elle estaujourd'hui encore beaucoup trop utilisée pentesteurs ( en
couple avec mimikatz, mais nous y reviendrons): la MS08-067 est une
faille historique dansle petit monde de la sécurité de part son
efficacités et sa stabilités. Nous allons aujourd'hui , disséquer la faille
en utiliseant un exploit et comprendre son fonctionnement.

Materiel :
~~~~~~~~~~

Pour pouvoir exploiter cette faille, il faut déjà une version de
windows compatible avec celle ci. Je préconise un windows XP Pro SP0 pour
commencer. Pour pouvoir suivre le fonctionnement de la faille, nous
allons utiliser OllyDbg http://www.ollydbg.de/, Enfin, nous
allons utiliser l’implémentation de l'exploit MS08-067 présente dans
Metasploit http://www.metasploit.com/.

1 - La faille :
~~~~~~~~~~~~~~~

La faille se trouve dans la lib `NETAPI32.dll`_. il s'agit de la fonction netpwPathCanonicalize().
Celle-ci appelle à son tour une autre fonction, et celle-ci utilise une fonction
non sécurisée de concaténation ( `wcscat`_ ).

Celle-ci tourne dans un des processus svchost ( qu'il faudra d'abord identifier).


2 - Reconnaissance :
~~~~~~~~~~~~~~~~~~~~

Nous allons déjà découvrir le procéssus hôte qu'il faut exploiter pour pouvoir
analyser le comportement de l'exploit.

.. code:: batch

  Microsoft Windows XP [Version 5.1.2600]
  (C) Copyright 1985-2001 Microsoft Corp.

  C:\Documents and Settings\pouete>tasklist /SVC

  Image Name                   PID Services
  ========================= ====== =============================================
  System Idle Process            0 N/A
  System                         4 N/A
  smss.exe                     368 N/A
  csrss.exe                    548 N/A
  winlogon.exe                 572 N/A
  services.exe                 652 Eventlog, PlugPlay
  lsass.exe                    664 PolicyAgent, ProtectedStorage, SamSs
  VBoxService.exe              820 VBoxService
  svchost.exe                  896 RpcSs
  svchost.exe                  996 AudioSrv, Browser, CryptSvc, Dhcp, dmserver,
                                 ERSvc, EventSystem,
                                 FastUserSwitchingCompatibility, helpsvc,
                                 lanmanserver, lanmanworkstation, Messenger,
                                 Netman, Nla, Schedule, seclogon, SENS,
                                 ShellHWDetection, srservice, TermService,
                                 Themes, TrkWks, uploadmgr, W32Time, winmgmt,
                                 WmdmPmSp, wuauserv, WZCSVC
  svchost.exe                 1064 Dnscache
  svchost.exe                 1088 LmHosts, RemoteRegistry, SSDPSRV, WebClient
  explorer.exe                1424 N/A
  spoolsv.exe                 1516 Spooler
  VBoxTray.exe                 212 N/A
  msmsgs.exe                   220 N/A
  cmd.exe                     1076 N/A
  tasklist.exe                1164 N/A
  wmiprvse.exe                1212 N/A


Nous nous intéresserons aux services réseaux. Ici c'est le PID 996 ( 0x3E4 ).
On lance OllyDbg pour s'attacher au processus n*3E4.
Il faut ensuite afficher les modules executables ( alt-e ) et trouver "NETAPI32.DLL" puis "view names" ( ctrl-n ).
Enfin, il faudra trouver la fonction NetpwPathCanonicalize().


.. _`wcscat`: https://msdn.microsoft.com/en-us/library/h1x0y282.aspx
.. _`NETAPI32.dll`: https://www.exploit-db.com/docs/320.pdf
