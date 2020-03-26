## core
git init

git status 

git log - historia commitow

git add <filename> - add to staging area from working area

git commit -m "description"  - gdy juz mamy w indexie (staged), mozna zapisac zmiany do repo

git checkout - checkout commitu lub brancha. Checkout - wyjmujemy z repo, wkladamy do working dir

git rm --cached <filename> - unstage file. File still exists in work dir

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

git checkout <branch> - checkout na brancha. 

<sha1> - checkout na commita. Wtedy bedzie detached head

**detached head** - normalnie head wskazuje na brancha. Robiac checkout na commita, 
head nie jest na branchu, jest bezposrednio na jakims commicie. To jest detached
mozna byc detached bedac np. na commicie, na ktorym jest master. checkout master naprawia

`Nie jest rekomendowane robienie commitow w detached, zostana stracone przy checkoucie na brancha`

git branch - lista lokalnych galezi

git branch <name> - tworzymy 

git checkout <name> - zmianimy

git checkout -b <name> - tworzymy i przechodzimy

git branch -d <name> - usuwanie zmergowanych branchy. -D - usuwa wszystkie

git branch -m <stary> <nowy> - rename

## Config
global/system/local configi

git config --<level> <param> <value>

git config --list

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

*`git hash object` create blob, 
*git cat-file -p/-t/-s hash` to read content of any git object
*git mktree`

echo "hello, git" | git hash-object --stdin

returns hash of file, with -w - creates git file inside .git/objects

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

add: przenosi untracked i modified do stage.

commit: staged->unmodified

edycja pliku - unmodified - modified

rm --cached: stage - unmodified

untrack - unmodified -> untracked