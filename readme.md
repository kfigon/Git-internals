## core
* `git init`
* `git status`
* `git log` - historia commitow
* `git add <filename>` - add to staging area from working area
* `git commit -m "description"`  - gdy juz mamy w indexie (staged), mozna zapisac zmiany do repo
* `git checkout` - checkout commitu lub brancha. Checkout - wyjmujemy z repo, wkladamy do working dir
* `git rm --cached <filename>` - unstage file. File still exists in work dir
* `git diff` - changes for unstaged files

## branching
branch - text reference to the commit. Branch to wrapper na commita

./git/refs/heads - all branches created in proj (with their last commit hashes)

branche nie sledza commitow na innych brancach. Trzeba zrobic checkout i wtedy mozna chodzic
branch pointer moves automatically after every new commit - branche sledza wlasne commity, branch jest najswiezszym commitem, pointer sie przesuwa (chyba ze do tylu z remote)
head - moze byc jeden. **Pointer do brancha (commita) - current snapshot of my project.**

checkout - `Zmiana heada`

referencja do obecnie checkoutowanego brancha/commita

.git/HEAD

default pointer is `ref: refs/heads/master`

`git checkout <branch>` - checkout na brancha. 
`<sha1>` - checkout na commita. Wtedy bedzie detached head

**detached head** - normalnie head wskazuje na brancha. Robiac checkout na commita, 
head nie jest na branchu, jest bezposrednio na jakims commicie. To jest detached
mozna byc detached bedac np. na commicie, na ktorym jest master. checkout master naprawia

`Nie jest rekomendowane robienie commitow w detached, zostana stracone przy checkoucie na brancha`

* `git branch` - lista lokalnych galezi
* `git branch <name>` - tworzymy 
* `git checkout <name>` - zmianimy
* `git checkout -b <name>` - tworzymy i przechodzimy
* `git branch -d <name>` - usuwanie zmergowanych branchy. -D - usuwa wszystkie
* `git branch -m <stary> <nowy>` - rename

## merging
Two approaches: 
* `fast-forward` - git przesuwa pointer heada. Mozliwy tylko gdy branch do ktorego megrujemy nie ma nowych commitow od czasu odejscia na brancha.
    * checkout docelowego (mastera) 
    * `git merge <feature-branch>` git przesunie mastera (i heada) na top feature-brancha 
    * mozna go juz usunac `git branch -d feature-branch`
* `three way merge` - git robi nowy commit ze zmianami. Gdy byly commity w docelowym. 
    * Nowy commit na podstawie ostatniego wspolnego commita (czyli skad odbilismy na feature), ostatni na feature i ostatni na docelowym 
    * 2 parentow nowego commita - ostatni top mastera i ostatni top feature
    * feature branch nietkniety. jak usuniemy brancha, to tylko branch jest usuniety, commity z feature zostaja
    * timeline - domyslnie bedzie chronologicznie. tortiose dziwnie pokazuje, `git log` pokazuje prawde

### Konflikty
* gdy w 3 way merge jest zmiana tej samej linijki
* `git merge --abort` do przerwania procesu, wracamy do heada
* wybrac co sie chce, zrobic stage i commita z fixem. **To bedzie ten nowy commit mergujacy branche**. Wystarczy `git add` i `git commit`, bedzie juz msg przygotowany (`git commit -a`)
* pojawi sie plik `.git/MERGE_HEAD` i w nim hash ostatniego commita z incomming brancha. `git merge --abort` usuwa ten plik
* w staging w tym czasie pojawiaja sie 3 wersje kazdego pliku - z wspolnego miejsca dla obu branchy, z heada i z feature (3 way)
* w working jest:
```
<<<<<<< HEAD
master commit
=======
change in br-3, hope to get conflict
>>>>>>> BR-3
```
## Remote
* `git clone`
* `git push` - wypycha zawartosc git repo do remote git repo
* `git pull` - remote -> working. pull = fetch+merge. `git merge FETCH_HEAD`  do working dir. jesli mozna to fast forward, jesli nie - zrobi 3way
    * **FETCH_HEAD** - `.git/FETCH_HEAD` sie pojawi samo.
* `git fetch` - zaciaga content remote git repo do lokalnego git repo. Nie laduje to w working dir. Nie updatuje working ani staging. Updatuje `./git/refs/heads/`

 **origin** - (domyslne) polaczenie local z remote.
 * `git remote` - listuje wszystkie polaczenia local z remotami (moze byc duzo, wtedy maja inne nazwy niz origin). `-v` wyspecyfikuje jakie operacje ida na jaki serwer (push i fetch moga byc na rozne)
 * `git remote show <nazwa serwera>` - serwer np. origin

 git nie odtwarza lokalnie wszystkich branchy.
 * `git branch -a` wylistuje wszystkie branche lokalne i zdalne. `-r` listuje tylko zdalne
    * `remotes/<nazwaSerwera>/branch` - nazwa to zazwyczaj origin

**tracking branch** - lokalny branch polaczony z remote branchem. git nie odtwarza lokalnie wszystkich branchy. branche tylko lokalne tez nie sa tracking.

* update trackinga - gdy zdalnie ktos usunie trakcing brancha, lokalnie zostaje. Update:
    * `git fetch`
    * `git remote update origin --prune`
    * `git branch -D <branch>` - usunie lokalnie

* `git branch -vv` - lista tracking branchy. lokanle ktore sa tez na remote
* `git checkout <nazwa>` - wycheckoutuje nawet zdalne branche
* `git push --set-upstream origin <nazwa brancha>` - push lokalnego brancha, ktorego nie ma na remote. Kolejne commity wystarczy push. W skrocie
`git push -u origin <branch>`


## Config
* `global/system/local configi`
* `git config --<level> <param> <value>`
* `git config --list`

## Git internals
inside **.git** - 3 pliki tekstowe 

config (local config), description, HEAD - current branch

git objects:
* blob - any file
* tree - info about directory
* commit - stored different version of file
* annotated tag - persistent text pointer to specific commit

git has it's own file system which uses objects

low level commands 

* `git hash object` create blob, 
* `git cat-file -p/-t/-s hash` to read content of any git object
* `git mktree`

`echo "hello, git" | git hash-object --stdin`

returns hash of file, with -w - creates git file inside `.git/objects`
if there are manby object, git compresses them and put into `./git/objects/pack`

`cat /.git/objects/pack/<name> | git unpack-objects`

hash - git trzyma pliki w swoim FS, identyfikuje je hashem. Folder name + file name = file sha1 hash (sha1 - 160bitow)
hashe sa trzymane w git w jsonie. kluczem jest poczatek hasha (directory), reszta jest wartoscia (plik)
hash zawsze zwraca to same dla tych samych danych, jest jednostronny,
nie odzyskamy oryginalu z hasha

blob nie trzyma nazwy pliku, tylko content, typ(blob) i rozmiar w blobie.

`content + type(blob) + size + \0 = hash`

tree - trzymaja pliki. Katalogi - polaczone drzewa

`content(nazwa) + type(tree) + size + \0 = hash`

hash pliku i nazwa pliku - laczymy w tree obj.
jedno tree moze zawierac wiele blobow

permissions in git objects - np. 040000 - directory, 100644 - regular file etc

jest polaczenie miedzy tree i blob i mamy mappin g FS gita a plik w naszym FS

te wszystkie wyzej rzeczy sa trzymane w **git repository**.

**working directory** - odwzorowanie w naszym FS.

pomiedzy nimi jest jeszcze **staging area (index)** odpowiada
za mapowanie w jedna i druga strone. tlumaczy hashe i strukture blobow i tree na faktycznie pliki.
Mandatory zeby przemieszczac pliki miedzy rejonami

`git ls-files -s` - pokazanie plikow w staging area.
Pliki przed skomitowaniem (do repo) lub przed checkoutem (do working dir)

`git read-tree <hash of tree>` - przenoszenie z repo do indexu

`git checkout-index -a` - przenoszenie indexu do working dir

**commit** - to samo co reszta. 
`content + type + size + \0 = hash`

Zawiera autora, nazwe, maila, opis i rodzica(optional)
i pointer (hash) do ktoregos tree. wrapper na tree.
przechowuje snapshoty (wersje) projektu.

File lifecycle:
* untracked - nowy plik, nie ma go w repo
* modified - plik jest w repo, inny niz working
* staged - zmiana jest w staging area
* unmodified - all in sync

------
* add: przenosi untracked i modified do stage.
* commit: staged->unmodified
* edycja pliku - unmodified - modified
* rm --cached: stage - unmodified
* untrack - unmodified -> untracked