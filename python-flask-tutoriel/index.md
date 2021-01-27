## Developper un site web avec Python Flask

- Installer Python. Un tutoriel interessant se trouve ici [RealPython](https://realpython.com/installing-python)
- Créer le repertoire de votre projet
- Entrer dans le repertoire et créer l'environnement virtuel (virtual env), comme suit :
```shell
python3 -m venv venv
```
Le deuxieme "venv" peut être n'importe quoi, par exemple le nom du projet

- Activer le virtual env :
```
source venv/bin/activate
```
Une fois l'environnement virtuel est créé et activé, on n'a plus besoin de specifier la version de python, comme python3 ou pip3
juste écrire python, pip et ça marchera

- Créer un fichier nommé requirements.txt et ajouter la ligne :
```
flask
```
- Créer un autre fichier requirements-dev.txt
```
-r requirements.txt
```
- Installer le framework
```
pip install flask
```
Si vous utiliser un outil de developpement comme PyCharm, il detecte automatiquement le contenu du fichier requirements.txt et installe le framework à votre place

- Maintenant, vous pouvez commencer à écrire votre premier programme, commencez par créer un fichier, par exemple app.py, à l'interieur de ce fichier, ecrire le code ci-dessous :

```python
import flask

app = flask.Flask(__name__)

@app.route('/')
def index():
  return "Bonjour le monde !"
app.run()
```

- On sauvegarde et on peut lancer le programme. Tout d'abord, il faut specifier deux variables d'environnement FLASK_APP et FLASK_ENV

```
export FLASK_APP=app #app correspond au nom de fichier qu'on a créé ci-dessus
export FLASK_ENV=development
```
- Maintenant, on lance le programme
```
flask run
```

Par défaut, Flask écoute sur le port 5000, si on souhaite specifier un autre si le 5000 est déjà utilisé
```
flask run -p 5001
```