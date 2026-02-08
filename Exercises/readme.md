# Observations personnelles : Développement de malware 

## 1. Introduction

Ce laboratoire vise à comprendre, les mécanismes internes utilisés par certains logiciels malveillants pour s’exécuter sur Windows. L’objectif est de voir comment ils manipulent la mémoire, les processus et les threads afin de mieux comprendre le fonctionnement du système d’exploitation et ses mécanismes de défense.

---

## 2. Environnement de travail

### Objectif
Mettre en place un environnement permettant la génération de payloads, le développement et l’exécution de programmes.

### Architecture
- **Génération du payload** : Kali Linux + `msfvenom`
- **Développement** : Windows 11, Visual Studio Code
- **Test** : Windows 11 (antivirus désactivé)
- **Langage** : Nim

Le shellcode utilisé a pour unique action le lancement de *notepad.exe*, afin de rester sur un comportement simple, inoffensif et observable.

La commande utilisé: 
- msfvenom -p windows/x64/exec CMD="C:\windows\system32\notepad.exe" EXITFUNC=thread -f nim

---

## 3. Exercice 1 : Loader local

### Objectif
Allouer de la mémoire, y copier un shellcode et l’exécuter dans le processus courant.

### Principe
- Allocation de mémoire exécutable avec VirtualAlloc()
- Copie du shellcode en mémoire 
- Exécution via la création d’un thread avec CreateThread()

Cet exercice introduit les bases de la gestion mémoire, des threads et de l’utilisation de l’API Win32.

### Observations
Le développement de malware est totalement nouveau pour moi, il m’a donc fallu me documenter pour comprendre ce que faisait les différentes méthodes, mais surtout sur le fonctionnement bas niveau d’un OS comme windows, sur la manière dont la mémoire était alloué et exécuté. Une fois la logique de base comprise, ce fus plus facile d'appréhender la suite.

---

## 4. Exercice 2 : Injection dans un processus distant

### Objectif
Exécuter le shellcode dans un processus déjà existant, contrairement au précédent qui l'éxecutait dans le process courant.

### Principe
- Chercher un processus déja lancé avec la commande Powershell Get-Process et copier le PID
- Ouverture d’un processus distant avec OpenProcess, cela crée le handle. Le handle est une référence qui permet d’intéragir avec les ressources windows (threads, processus,clés de registre, ect…). 
- Allocation de mémoire dans son espace d’adressage avec VirtualAllocEx(Alloue une région mémoire dans l’espace d’adressage d’un processus distant, contrairement à VirtualAlloc qui le met dans la mémoire local)
- Écriture du shellcode avec WriteProcessMemory()
- Exécution via un thread distant avec CreateRemoteThread (crée le thread dans un autre processus)


---

## 5. Exercice 3 : Obfuscation et évasion

### Objectif
Rendre le shellcode moins détectable en le chiffrant via un XOR.

### Principe
- Chiffrement simple du shellcode (XOR), une clé de taille d'un byte, on fait un xor sur chaque byte du shellcode
- Déchiffrement en mémoire avant l’injection suivant le même principe (XOR)

Le payload continue de s’exécuter même après la fin du programme injecteur.

---

## 6. Bonus

### Bonus 1: Exécution sans thread
Le shellcode est exécuté directement comme une fonction du programme NIM, sans créer de thread, ce qui veut aussi dire en ayant moins recours à l'API Win32. Cela démontre qu’exécuter du code revient simplement à appeler une adresse mémoire.

### Bonus 2: Injection autonome
Le programme gère lui-même la recherche du processus cible et le crée si il n'éxiste pas, sans PID codé en dur mais avec un nom en entré. Contrairement à l'exo 2 ou on présupposait que le processus cible tournait déja (plantage si ce n'était pas le cas). 

### Bonus 3: API NT natives
Utilisation directe des appels NT (`NtOpenProcess`, `NtAllocateVirtualMemory`, `NtWriteVirtualMemory`, `NtCreateThreadEx`, `NtClose`) au lieu des API Win32 équivalentes. L'API NT étant plus proche du noyau, elle est historiquement filtré, ce fut surtout un changement de couche, pas de logique car le programme faisait exactement la même chose fonctionnellement que l’exercice 3

---

## 7. Conclusion

Ces exercices permettent de comprendre progressivement comment Windows gère l’exécution du code, de la mémoire et des processus. Ils montrent qu’un programme est avant tout une suite de bytes placée en mémoire et exécutée par le processeur.

La progression (exécution locale, injection distante, gestion dynamique des processus, puis appels NT natifs) met en évidence que les API Win32 sont des abstractions construites sur des mécanismes bas niveau.

Ce fut un apprentissage enrichissant qui m'a permis d'apprendre sur le fonctionnement plus bas niveau des OS et d'avoir une introduction aux reverse engineering.

En résumé, ces exercices m’ont appris à lire et comprendre de la documentation bas niveau, manipuler des structures système, et accepter de sortir du cadre “safe” et abstrait des langages haut niveau.
