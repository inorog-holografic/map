Aplicația task manager este un CLI care face posibilă gestionarea task-urilor cu priorități,
termene și tags. Am folosit structuri de date : heap (toate tasks sunt stocate într-un heap 
pentru sortare automată după prioritate și termen.); structuri simple(Task cu campurile id, titlu,
descriere, rpioritate, termen, tags, complet).
Din punct de vedere al funcțiilor, avem : 
1. add_task – adaugă un task nou în heap.
     parametri: titlu, descriere, prioritate de la 1 la 5, termen yyyy-mm-dd, tags.
     actualizează heapul folosind heap_up pentru a păstra ordinea.
     salvează automat taskurile în tasks.json.
     se creează o variabilă locală de tip Task pe stivă. t.id=id_curent++ - atribuie id-ul curent (încărcat din json sau pornit de la 1) și incrementeaza pt. următorul task. Cu strcpy se transferă datele primite ca argumente, inputul nu trebuie să depășească MAX_TEXT care are 256 caractere. t,complet=0 - orice task nou este setat automat ca incomplet/TODO. heap.taskuri[heap.nr] = t - Task-ul este pus pe ultima poziție liberă din array-ul heap-ului
heap.nr - indicator de dimensiune și, în același timp, ca index pentru următorul element gol. 
heap_up(heap.nr) - restabilim ordinea; heap.nr++ - incrementăm contorul; prioritate_text(t.prioritate) - transformăm cifra primitp în text 

2. list_tasks – listează toate task-urile într-un tabel.
     afișează ID, titlu, prioritate cu text descriptiv, termen, status complet/incomplet.
     sortarea e automată datorită heap-ului.
     for (int i = 0; i < heap.nr; i++) : arcurge array-ul taskuri de la indexul 0 la nr-1
     t.complet ? "OK" : "TODO": În loc să afișeze cifra 0 sau 1 din structură, programul                verifică valoarea pe loc
   
3. complete_task – marchează un task ca complet după ID.
   căutarea: for (int i = 0; i < heap.nr; i++) – heap-ul este sortat după prioritate, nu după ID, programul trebuie să verifice fiecare element până găsește ID-ul cerut
   modificarea: heap.taskuri[i].complet = 1; – odată găsit, bitul de stare se schimbă din 0 în 1

4. edit_task – modifică prioritatea unui task și rearanjează heap-ul.
     identificarea : La fel ca la complete_task, parcurge arra-ul până găsește task-ul cu id-ul respectiv
     actualizarea : heap.taskuri[i].prioritate = prioritate; – se suprascrie valoarea veche cu cea nouă primită din CLI
     rearanjarea : * heap_down(i); și heap_up(i); – Aceasta este partea cea mai importantă. Când schimbi prioritatea, task-ul respectiv poate trebuie să urce sau trebuie să coboare, în funcție de schimbările în importanță
   
5. filtre :
    filter_overdue – afișează task-urile cu termen depășit.
             condiție dublă : if (!heap.taskuri[i].complet && este_overdue(...))
             logica : verifică doar task-urile care nu sunt gata (!complet).
             funcția este_overdue : apelează azi(curent) pentru a lua data sistemului și                        folosește strcmp. dacă data termenului este mai mică alfabetic decât data de azi,                task-ul este marcat ca restanță.
    filter_today – afișează task-urile cu termenul astăzi.
             logica : strcmp(termen, curent) == 0.
             funcționare : compară string-ul de termen direct cu string-ul generat de funcția                    azi. Dacă sunt identice caracter cu caracter, task-ul este afișat ca fiind                       pentru ziua curentă.
    filter_week – afișează task-urile cu termenul în următoarele 7 zile.
               logica : int z = zile_ramase(termen); return z >= 0 && z <= 7;
               funcționare : transformă data din string în secunde prin mktime, calculează                        diferența față de secunda actuală a sistemului, împarte la 86400 pentru a afla                   zilele

6. stats – afișează statistici generale: total tasks, câte sunt complete, incomplete, overdue.
          int total = heap.nr : numărul total de task-uri existente în heap în acel moment
          for (int i = 0; i < heap.nr; i++) : parcurgere liniară, primul-ultimuk
          if (heap.taskuri[i].complet) comp++; : verifică flag-ul complet. dacă este 1,                          incrementează contorul pentru task-uri terminate
          if (este_overdue(heap.taskuri[i].termen)) over++; : apelează funcția temporală care                     compară data curentă a sistemului cu termen. Dacă data a trecut, incrementează                   contorul de restanțe
7. json :
     save_json – salvează task-urile curente în tasks.json. Scrie manual caracterele {, [ și           numele câmpurilor. Este un export hardcoded care transformă structura din memorie într-un        format text pe disc.
     load_json – poate încărca taskuri existente la pornirea aplicației. Folosește un format           specific în sscanf : %[^"] . ^" îi spune programului să citească tot textul până când dă         de ghilimele. Așa poate să citească titluri sau taguri care au spații în ele (un exemplu         ar fi : "Cumparaturi saptamanale"), astfel nu se oprește la primul spațiu.
      La finalul funcției load_json, există for care pleacă de la jumătatea heapului în sus și         apelează heap_down. Astfel, dacă fișierul json a fost modificat manual, programul repară         ordinea priorităților imediat ce pornește.

8. compara(Task a, Task b) : funcția returnează "adevarat" dacă a este mai important decât b. Verifică întâi a.prioritate != b.prioritate. Dacă sunt diferite, cel cu cifra mai mare câștigă. Dacă sunt egale, folosește strcmp(a.termen, b.termen) < 0 pentru a pune data mai mică (calendaristic mai apropiată) în față.

9. heap_up, heap_down – mențin structura heap-ului pentru prioritizare.
   heap_up(int idx) : Când adaugi un task nou la final (heap.nr), acesta urcă în sus.  Verifică        părintele la (idx - 1) / 2. Dacă task-ul nou are prioritate mai mare (folosind funcția           compara), face swap cu părintele până când îi găsește locul corect.
   heap_down(int idx) : Folosită la editare. Dacă modifici prioritatea și task-ul devine mai           neimportant, acesta trebuie să coboare. Verifică copiii (stânga 2*idx + 1, dreapta 2*idx +       2), găsește cel mai prioritar copil și face schimb cu el.

10. temporale :
      azi() – returnează data curentă.
      este_overdue(), este_today(), zile_ramase() – verifică termenele și calculează timpul
rămas până la deadline.
      - azi(char *buf) : Folosește time.h pentru a extrage ora sistemului și o formatează strict ca yyyy-mm-dd folosind sprintf.
      - zile_ramase(char *termen) : se face conversie manuală. Ia string-ul, îl desface cu sscanf în an, lună, zi, și le pune în structura struct tm. mktime transformă totul în secunde totale (Unix time). Scade timpul curent din deadline și împarte la 86400 (secunde într-o zi) pentru a obține numărul de zile.
     - este_week : Pur și simplu apelează zile_ramase și verifică dacă rezultatul este între 0 și 7

MAIN : Prima acțiune realizată este inițializarea numărului de task-uri la zero, urmată de apelul funcției load_json. după pregătirea datelor, main verifică dacă utilizatorul a introdus cel puțin un argument după numele programului : dacă nu e ok afișeaă mesaj și oprește execuția programului pentru a evita erori. Dacă există argumente, programul compară primul argument, argv[1], cu o serie de cuvinte cheie precum add, list sau edit, folosind funcția strcmp. funcția get_arg caută în lista de argumente valorile specifice pentru prioritate, termen limită, descriere și etichete. dacă lipsesc informații esențiale, cum ar fi titlul sau prioritatea, programul afișează un avertisment și se închide. altfe, se apelează funcția add_task pentru a introduce în heap și apoi save_json pentru a scrie modificarea pe disc. ramificarea se cotninuă într-un mod similar; pentru list se afișează tabelul, iar pentru complete sau edit, main extrage id-ul necesar, face modificarea și salvează starea nouă a fișierului. în cazul filtrării, funcția verifică flags specifice de timp(--overude, --today) și indică datele către funcțiile corespunzătoare pentru afișare. la finalul programului, dacă nicio comandp cunoscută NU se potrivește cu ceea ce a introdus userul, din main se trimite un mesaj de eroare, apoi se face return 0, astfel se închide aplicația.
        Comenzi CLI :
get_arg : Rulează prin vectorul de argumente argv și caută string-ul dorit (ex: --id). Dacă îl găsește, returnează pointerul către poziția următoare (argv[i + 1]), adică valoarea căutată.

main : Verifică argv[1] (comanda: add, list, edit, etc.) și apelează funcția corespunzătoare, apoi termină execuția cu save_json pentru a nu pierde nicio modificare.  
       
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




