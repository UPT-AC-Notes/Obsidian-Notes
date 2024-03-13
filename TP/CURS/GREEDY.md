# Lab11: Greedy
Metoda Greedy este o tehnică de programare care încearcă să găsească soluția optimă pentru o problemă prin alegerea celei mai bune opțiuni în fiecare etapă a algoritmului, fără a lua în considerare impactul acestei decizii asupra soluției finale. Optimul local NU poate garanta intotdeauna optimul global.

Algoritmul Greedy începe cu o soluție parțială și adaugă continuu elemente la aceasta, alegând mereu cea mai bună opțiune disponibilă în acel moment. Fiecare decizie luată este finală și nu se ia în considerare posibilitatea revenirii la o decizie anterioară sau la o altă soluție parțială. În general, metoda Greedy se folosește atunci când problema se poate împărți în subprobleme independente și când se dorește o soluție rapidă.

Într-un exemplu simplu, putem să ne gândim la problema găsirii celei mai scurte căi între două puncte pe o hartă. Algoritmul Greedy ar putea alege mereu cel mai apropiat punct disponibil, până când ajunge la destinație. Totuși, această metodă nu garantează întotdeauna soluția optimă, deoarece există situații în care alegerea celei mai bune opțiuni în fiecare etapă nu duce la soluția globală optimă.

Pentru a folosi metoda Greedy în programare, trebuie să se definească un criteriu de selecție a celor mai bune opțiuni disponibile, care să asigure atingerea obiectivului final. De asemenea, trebuie să se testeze algoritmul pe diferite date de intrare pentru a se asigura că funcționează corect și că nu există cazuri care să ducă la o soluție suboptimală.

Problema spectacolelor
Problema spectacolelor (codul se gaseste pe CV) este un exemplu clasic de aplicare a metodei Greedy în programare și constă în a găsi cel mai mare număr de spectacole care pot fi planificate într-o sală de spectacole, astfel încât să nu existe nicio suprapunere între ele. Drept intrare se cunosc orele de incepere si incheiere a fiecarui spectacol.

Pentru a rezolva această problemă cu ajutorul metodei Greedy, putem proceda astfel:

1. Sortăm toate spectacolele în funcție de ora de încheiere a fiecărui spectacol. Astfel, vom avea spectacolele cu ora de încheiere mai mică mai devreme în listă.

2. Inițializăm un contor pentru numărul de spectacole planificate, pe care îl setăm la 1, deoarece primul spectacol din listă va fi întotdeauna inclus în programul final.

3. Parcurgem lista de spectacole începând cu al doilea spectacol și verificăm dacă ora de început a acestuia este după ora de încheiere a ultimului spectacol programat.

4. Dacă este așa, adăugăm acest spectacol la programul final și incrementăm contorul pentru numărul de spectacole planificate.

5. Dacă ora de început a spectacolului este înaintea orei de încheiere a ultimului spectacol programat, nu îl putem include în programul final și trecem la următorul spectacol din listă.

6. Parcurgem astfel toate spectacolele din listă și obținem astfel numărul maxim de spectacole pe care le putem programa fără suprapunere.

Deși această metodă poate să nu producă întotdeauna soluția optimă, ea este o soluție aproximativă rapidă (ce complexitate?) și simplă care poate fi utilizată în multe situații practice unde o resursa unica trebuie partajata.

Aplicatii
1. Se dă o colectie cu n numere întregi (varainta: numere naturale). Determinați cel mai mare număr care poate fi scris ca produs de două elemente ale șirului. La intrare se va citi un numar n si apoi n numere intregi/naturale separate prin minim un spatiu, iar la iesire se va scrie un singur numar care este rezultatul solicitat.

2. Se dau trei numere naturale n a b, 1 ≤ a < b < n. Să se determine o modalitate de a-l scrie pe n ca sumă de termeni egali cu a sau b în care numărul de termeni egali cu a este maxim. La intrare se vor citi n a si b, in aceasta ordine, iar la iesire se va scrie valoarea numarului de aparitii ale lui a si ale lui b, in aceasta ordine, separate prin minim un spatiu.

3. Cunoscând timpul necesar pentru analizarea fiecărui proiect, scrieţi un program care determină ordinea în care vor fi analizate proiectele, astfel încât timpul mediu de aşteptare pentru investitori să fie minim. Drept date de intrare se cunosc n (numarul de proiecte) si o lista de valori numerice naturale reprezentand timpii de evaluare pentru fiecare proiect (in unitati abstracte de timp)

4. Se dă un șir de n numere naturale. Șirul poate fi partiționat în mai multe moduri într-un număr de subșiruri strict crescătoare. De exemplu, șirul 4 6 2 5 8 1 3 7 poate fi partiționat astfel: 4 6 8 (primul subșir), 2 5 7 (al doilea) și 1 3 (al treilea). O altă modalitate este formând patru subșiruri: 4 5 7, 6 8, 2 3 și 1.

5 (*). Se consideră o tablă de şah cu n linii şi m coloane. La o poziţie dată se află un cal de şah, acesta putându-se deplasa pe tablă în modul specific acestei piese de şah (în L).

Să se determine o modalitate de parcurgere a tablei de către calul dat, astfel încât acesta să nu treacă de două ori prin aceeaşi poziţie. Parcurgerea constă într-o matrice cu n linii și m coloane, fiecare element al matricei având valoarea pasului la care se ajunge în acel element, sau 0, dacă nu s-a ajuns.

Referinte

[1] https://www.geeksforgeeks.org/warnsdorffs-algorithm-knights-tour-problem/

[2] http://warnsdorff.com/

6. Se citesc de la intrarea standard, 3 numere naturale S, n si e cu urmatoarele semnificatii: S este o suma de bani care trebuie platita folosind bancnote care au valori puterile lui e, de la e la 0, la e la n. Se se afiseze modalitatea de plata (câte bancnote de ce tip) folosind un numar minim de bancnote de tipurile precizate si sa se afiseze la final numarul total de bancnote folosite.