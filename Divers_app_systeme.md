##[Bash Broken input](#Bash-Broken-input)


#<a name="Bash-Broken-input"></a>
Dans un script bash, on peut évaluer des choses même si ces choses là sont dans des ' .

Basiquement, lorsqu'on met une variable comme FOO=42 et qu'on l'invoque avec '$FOO', la réponse sera : $FOO et non pas 42.

Par contre, bash évalue et expanse ce quil y a à l'intérieur de tableau.

Donc par exemple, si on fait FOO="x[$(echo toto)]" et qu'on fait '$FOO' on aura : toto.

Avec python 2.7.17 on peut acceder a n'importe quelle fonction dans le script, ou bien importer une fonction via __import__:

__import__('sys').stdout.write(open("flag.txt", 'r').read())
__import__('os').system('cat flag.txt> /tmp/flag.txt')

=> GOTO flag.txt

Ou bien :

#!/usr/bin/python2

import sys

def ultrasecret():
    open("supersecret", 'r').read()
    sys.exit(1)

try:
    p = input("Please enter password : ")
    

Please enter password : ultrasecret()

PYTHONINSPECT=YesMyLord ./xx
Please enter password : blabla !
Traceback (most recent call last):
<SNIP>
>>> f = open("flag")
>>> print f.readlines()
['...']


Perl command Injection :

The open() function can be used to pass a command :
# ex : “| cat flag.txt |”
