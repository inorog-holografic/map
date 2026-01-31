Aplicația task manager este un CLI care face posibilă gestionarea task-urilor cu priorități,
termene și tags. Am folosit structuri de date : heap (toate tasks sunt stocate într-un heap 
pentru sortare automată după prioritate și termen.); structuri simple(Task cu campurile id, titlu,
descriere, rpioritate, termen, tags, complet).
Din punct de vedere al funcțiilor, avem : 
1. add_task – adaugă un task nou în heap.
     parametri: titlu, descriere, prioritate de la 1 la 5, termen yyyy-mm-dd, tags.
     actualizează heapul folosind heap_up pentru a păstra ordinea.
     salvează automat taskurile în tasks.json.
2. list_tasks – listează toate task-urile într-un tabel.
     afișează ID, titlu, prioritate cu text descriptiv, termen, status complet/incomplet.
     sortarea e automată datorită heap-ului.
3. complete_task – marchează un task ca complet după ID.
4. edit_task – modifică prioritatea unui task și rearanjează heap-ul.
5. filtre :
    filter_overdue – afișează task-urile cu termen depășit.
    filter_today – afișează task-urile cu termenul astăzi.
    filter_week – afișează task-urile cu termenul în următoarele 7 zile.
6. stats – afișează statistici generale: total tasks, câte sunt complete, incomplete, overdue.
7. json :
     save_json – salvează task-urile curente în tasks.json.
     load_json – poate încărca taskuri existente la pornirea aplicației.
8. heap_up, heap_down – mențin structura heap-ului pentru prioritizare.
9. temporale :
      azi() – returnează data curentă.
      este_overdue(), este_today(), zile_ramase() – verifică termenele și calculează timpul
rămas până la deadline.
Aplicația poate fi rulată direct din terminal sau într-un container Docker. În Docker,
imaginea construită conține codul compilat și toate dependențele, astfel încât rularea
se face cu o singură comandă docker run urmată de argumentele CLI corespunzătoare.

pașii pentru a rula din docker :
1. docker pull carolinaszoke/task_manager:1.0
2. structura generala a comenzii : docker run --rm -it carolinaszoke/task_manager:1.0 <comanda> [argumente]

3. EXEMPLE :
   adaugare task : (sudo) docker run --rm -it carolinaszoke/task_manager:1.0 add "Proiect ATAD" --priority 5 --due 2026-02-05 --desc "Test CLI" --tags "facultate,urgent"
   listare : docker run --rm -it carolinaszoke/task_manager:1.0 list --sorted priority
   complet : docker run --rm -it carolinaszoke/task_manager:1.0 complete --id 1
   prioritate : docker run --rm -it carolinaszoke/task_manager:1.0 edit --id 1 --priority 4
   filtare :
     docker run --rm -it carolinaszoke/task_manager:1.0 filter --overdue
     docker run --rm -it carolinaszoke/task_manager:1.0 filter --today
     docker run --rm -it carolinaszoke/task_manager:1.0 filter --week
   statistici : docker run --rm -it carolinaszoke/task_manager:1.0 stats
(--rm sterge containerul automat după ce se inchide;
--it : interactive+terminal permite utilizatorului să vadă outputul CLI)




