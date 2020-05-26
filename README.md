# UVRG_Projektna_1
V sklopu projekta boste pri tem predmetu implementirali 3D trilateracijo, torej pozicioniranje v prostoru glede na neke referenčne oddajnike.
## 1. del
Najprej bomo naredili osnovno trilateracijo, ki jo bomo kasneje nadgrajevali.

Pri prvem delu te naloge boste dobili tri referenčne bazne postaje, kjer za vsako veste njeno X, Y in Z koordinato ter oddaljenost od nje. Cilj naloge je s temi podatki dobiti vaš položaj v 3D prostoru. Sama natančnost položaja pa je odvisna od "kvalitete" podatkov, torej kako natančno so izmerjene razdalje kot tudi sami znani položaji postaj. Kako šum vpliva na izračun položaja bomo videli v projektni nalogi - 2. del.

Aplikacija je lahko konzolna, vendar naj omogoča vnos treh referenčnih postaj ter razdalj od njih. To vajo naredi vsak posameznik sam, potem v drugi nalogi pa boste vzeli implementacijo od tistega člana skupine, ki jo je najbolje implementiral in potem boste tisto verzijo nadgrajevali. To vajo naredi in odda vsak posameznik skupine!

Pozicije referenčnih postaj:
P1 = (x1, y1, z1)
P2 = (x2, y2, z2)
P3 = (x3, y3, z3)

Ter tri razdalje:
r1, r2, r3

Problem si želimo računsko poenostaviti in bi si želeli, da točke P1, P2 in P3 vse ležijo v isti ravnini. Žal pa to ni vedno res, saj so lahko njihove pozicije naključno razporejene po prostoru. K sreči lahko poljubno ravnino točno določimo s tremi točkami. Po čudnem naključju imamo v nalogi ravno tri točke! To pomeni da lahko menjamo naš kartezični koordinatni sistem za takšnega, kjer bodo vse točke ležale v isti ravnini oziroma kjer bodo vse točke imele koordinato z = 0.

Da menjamo koordinatni sistem vzemimo za začetek eno izmed točk, na primer P1, kot izhodišče novega koordinatnega sistema in jo odštejmo od P1, P2 in P3.

P1' = (0, 0, 0)
P2' = (x2 - x1 ,y2 - y1, z2 - z1)
P3' = (x3 - x1, y3 - y1, z3 - z1)

Nato vzamemo vektor V1 = P2' - P1' in ga normaliziramo, da dobimo novo os X, Xn = V1 / |V1|. Manjkata nam še ortogonalni osi Y in Z, do katerih pa lahko pridemo s pomočjo 3D vektorskega produkta. Za začetek vzemimo kot začasno os Y kar ne ortogonalni vektor V2 = P3' - P1' = P3'. Tako V1 kot V2 ležita v isti ravnini. Z vektorskim produktom lahko za ravnino, ki ju razpenjata vektorja V1 in V2 poiščemo njej ortogonalni vektor oz. normalo, Zn = (V1 x V2) / |V1 x V2|. Potrebno je še določiti os Y, kar pa sedaj lahko naredimo kar z osjo Xn in Zn, Yn = Xn x Zn. Ta korak predstavlja poenostavljen Gram–Schmidtov postopek ortonormalizacije baznih vektorjev.

P1', P2' in P3' lahko sedaj zapišemo z novimi osmi Xn, Yn in Zn kot:
P1' = (0, 0, 0)
P2' = (d, 0, 0)
P3' = (i, j, 0)
kjer je d = Xn · V1 = |V1|, i = Xn · V2 in j = Yn · V2

Sedaj nas zanima naša pozicija X, Y in Z. Na voljo imamo sledeče enačbe sfer:
r12 = X2 + Y2 + Z2
r22 = (X - d)2 + Y2 + Z2
r32 = (X - i)2 + (Y - j)2 + Z2

Če sistem enačb preuredimo, pridemo do naslednje rešitve:
X = (r12 - r22 + d2) / 2d
Y = (r12 - r32 + i2 + j2) / 2j - (i / j) * X
z = √(|r12 - X2 - Y2|)


Ne smemo pozabiti, da smo še vedno v našem spremenjenem kartezičnim prostoru! Da se vrnemo nazaj v "normalne" koordinate je potreben še en korak:
K1 = P1 + X * Xn + Y * Yn + z * Zn
K2 = P1 + X * Xn + Y * Yn - z * Zn

Dodatna pomoč pri nalogi
Viri:
https://en.wikipedia.org/wiki/Trilateration
https://en.wikipedia.org/wiki/Gram%E2%80%93Schmidt_process

Vektorski produkt v 3D:
Za vektorja v1 = (x1 ,y1, z1), v2 = (x2, y2, z2):
v1 x v2 = (y1 * z2 - z1 * y2, z1 * x2 - x1 * z2, x1 * y2 - y1 * x2)
Zelo verjetno, da je funkcija že implementirana v standardnih programskih knjižnicah.

Dva primera nad katerimi lahko preverite pravilno delovanje:

### Primer 1:
P1 = (2, 2, 0), P2 = (3, 3, 0), P3 = (1, 4, 0)
r1 = 1, r2 = 1, r3 = 1.4142
Pravilna rešitev:
P = (2, 3, 0)

### Primer 2:
P1 = (2, 1, 0) , P2 = (4, 3, 0), P3 = (4, 4, 1)
r1 = 2, r2 = 2, r3 = 2.449
Pravilna rešitev:
P = (2, 3, 0)

## 2. del

V prvem delu naloge ste implementirali trilateracijo s pomočjo treh baznih postaj P1, P2 in P3.

Kot že omenjeno, to povzroča težave, saj ne vemo naše točne pozicije. Kadar so bazne postaje na isti višini nam to prestavlja dvoumnost v naši višini, kadar pa so postaje naključno razpršene po prostoru pa se ta dvoumnost preslika na X, Y in Z koordinate. Res je, da je dvoumna koordinata samo Z, a smo v prvem delu menjali koordinatni sistem in je lahko bazni vektor za Z os poljubno usmerjen.

Težavo lahko odpravimo, če imamo še četrto referenčno postajo ter oddaljenost nje:
P4 = (x4, y4, z4)
r4

To točko prestavimo v naš spremenjen koordinatni sistem, podobno kot v prvem delu tukaj projiciramo točko P4 na naše bazne vektorje, da dobimo za vsako koordinatno pripadajoče skalarje:
a = (P4 - P1) * Xn
b = (P4 - P1) * Yn
c = (P4 - P1) * Zn

Dobimo sledeči položaj ter enačbo oddaljenosti:
P4' = (a, b, c)
r42 = (X - a)2 + (Y - b)2 + (Z - c)2

Da dobimo točno vrednost Z rešimo naslednjo enačbo:
r42 - r12 = (X - a)2 + (Y - b)2 + (Z - c)2 - X2 - Y2 - Z2

in dobimo:
Z = (r12 - r42 + a2 + b2 + c2) / 2c - (a / c) * X - (b / c) * Y

Ne pozabite, da se je potrebno prestaviti nazaj v začetni koordinatni sistem.

V realnih okoljih se pogoji praviloma vedno spreminjajo in prihaja do tega, da se nam število postaj spreminja skozi čas. Predpostavimo, da v danem trenutno vidimo N postaj, kjer je 3 <= N <= 10.

Implementirajte sledeče obnašanje:
- če vidite samo 3 postaje, potem je položaj dvoumen. V primeru, da to ni prvi izračun položaja, izberite tisto izračunano vrednost, ki je najbližja nazadnje izračunanemu položaju,
- kadar vidite točno 4 postaje je pozicija natančno znana,
- kadar vidite več kot 4 postaje boste izračunali vaš položaj večkrat in rezultat povprečili. Izračun položaja naredite tako, da izberete točno štiri naključne postaje, ampak nikoli ne izberite istega nabora postaj dvakrat! Izračun ponovite N-krat. Ko imate izračunane vse položaje jih povprečite:
P = (P0 + P1 + ... + Pn) / N

in izračunajte napako RMSE (Root Mean Square Error).
err =√((∑(|Pi - P|2) / N)

Izpišite povprečen položaj ter ocenjeno napako.

### Primer 1:
Vidimo točno 4 postaje in meritve razdalje so brez napake.
P1 = (2, 1, 0), P2 = (4, 3, 0), P3 = (1, 4, 0), P4 = (2, 3, 4)
r1 = 1.5, r2 = 2.0616, r3 = 2.6926, r4 = 3.2016
Pravilna rešitev:
P = (2.5, 2, 1)

### Primer 2:
Kot primer, vidimo 6 postaj kjer imajo meritve 5% napako.
P1 = (2, 1, 0), P2 = (4, 3, 0), P3 = (3, 3, 1), P4 = (2, 3, 4), P5 = (5, 5, 2), P6 = (3, 8, 3)
r1 = 0.9661, r2 = 2.2039, r3 = 1.7243, r4 = 4.0825, r5 = 4.7003, r6 = 6.7794
Spodaj imamo šest možnih rešitev, pozicije od R1 do R6. Za R1 izberemo meritve in pozicije (1, 2, 3, 4), za R2 (2, 3, 4, 5), za R3 (3, 4, 5, 6), za R4 (4, 5, 6, 1), za R5 (5, 6, 1, 2) in za R6 (6, 1, 2, 3).
R1 = (2.1636, 1.8555, 0.1056)
R2 = (2.1636, 1.7539, 0.1056)
R3 = (1.9650, 1.9855, 0.0394)
R4 = (1.9657, 1.9856, 0.0405)
R5 = (1.9807, 2.0383, -0.0874)
R6 = (1.9909, 2.0281, -0.0671)

Povprečna točka:
AVG_R = (2.0382, 1.9411, 0.0227), RMSE pa je 0.1557

Rezultat povprečnega položaja (posledično tudi RMSE) bo seveda drugačen, kadar izberemo drug nabor točk! Pravilna rešitev je bila v tem primeru (2, 2, 0), tako, da se s povprečenjem približamo našemu "realnemu" položaju.
