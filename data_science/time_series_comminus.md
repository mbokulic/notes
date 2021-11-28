Ne mozes predvidjati ponasanje pojedinca vec predvidjas segmente. Dakle treba naci segmente koji imaju slicne obrasce u vremenskim serijama.

Segmentiras po staticnim varijablama. Dakle uzmes kros-sekciju ponasanja (neki period velicine x) i izracunas staticne varijable na toj kros-sekciji. Tipa broj oklada po mjesec dana, prosjecna velicina oklade, prosjecni koeficijent, na sto se kladi, itd. Generiras cca 40tak segmenata.
 - nismo pricali o tome kako segmentirati. Pretpostavljam klaster analiza.

Onda iterativno: vidjeti da li ti segmenti daju konzistentne vremenske serije. Ako ne, probat druge segmente. Jos ne znam kako se usporedjuju vremenske serije jedna sa drugom. Pitao sam da li obrazac ili apsolutne vrijednosti. Navodno se gleda obrazac i postoje tipicni nacini.
 - normalizacija pa euclidean distance ili dynamic time warping
 - ima i drugih varijanti, zanimljivo mi se cini ono gdje se gledaju staticne varijable (Hyndman clanak)

Nakon sto nadjes segmente sa konzistentnim vremenskim serijama, gledas kakvo ponasanje vodi do prestanka kladjenja. Trazis simptome prestanka (kako se ljudi ponasaju tik prije prestanka)
 - ovo je ja mislim klasicni churn pristup gdje se ljudi podijele u churn-ere i ne-churnere te gledas staticne prediktore

e da bi nasao uzroke prestanka (sto se desilo prije prestanka). Drugim rijecima, trazis proksimalne prediktore prestanka da bi kasnije nasao raniji uzrok. Nije mi jasno kako to.
 - ovo mi jos uvijek nije jasno

# an idea of how to proceed

## segmenting on static variables

Segmentirati po staticnim varijablama. Ova segmentacija mora proizvesti grupe sa konzistentnim vremenskim serijama zato sto ce se na tim ts traziti simptomi i uzroci churn-a.

Kako naci staticni period

 - odbaciti ljude koji se nisu kladili bar N puta i bar M mjeseci. Za obje skupine imamo premalo podataka, a i oni nisu ionako bitni
 - mozemo se i ograniciti samo na prvih 80% prihoda?
 - uzeti X mjeseci najzesceg kladjenja (to je zato sto zelis uzeti nesto stabilno. U svakom slucaju ne zelis dodirivati formativni period i churn period).
 - mozda za neke varijable ima smisla gledati cijeli period? U kladjenju postoji zestoka sezonalnost zbog liga i ako gledam samo X mjeseci mozda propustim vidjet neki od sportova

Koja metodologija:

 - hijerarhijska klaster analiza (hijerarhijska jer nemam pojma koliki k uzeti, bolje gledati one dendrograme)
 - distance metrika moze euklidska distanca, nije me briga

Koje varijable? One koje mogu utjecati na obrazac kladjenja (frekvenciju, sezonalitet, tipove oklada), reakcije na ishode oklade (zasto igra, koliko para ima za izgubiti), razloge churna (izgubio pare, dosadilo, nasao bolju kladionicu)...

 - velicina oklade
 - sportovi na koje se kladi (uzeti par najcescih kao kategoricke varijable)
 - broj parova
 - prosjecni koeficijent
 - broj oklada

### similarity between ts
Uspjesnost segmentacije mjeri se po tome koliko su ts svakog segmenta slicne. Za ts nejednake duljine uspio sam naci ove opcije:

 - dynamic time warping: zar je to stvarno dobro ako se ts jako razlikuju po duljini? I morao bih usporediti svaku sa svakom... puno procesiranja. Mozda usporedjivati sa prosjekom.
 - pretvaranje ts u staticne varijable (Hyndman). Ovo mi se trenutno cini najsmislenije: da se od slicnih ts ocekuje da imaju slicni sezonalitet, kaoticnost, trend, itd.
 - moja ideja: izracunati prosjecnu ts pa gledati varijancu. Missing values odbaciti. Ako su jako razlicitih duljina opet nema puno smisla. Onda ce dulji ts imati veci utjecaj. Mozda nekako penalizirati NAs, ali kako.
 - longest common subsequence: tendirati ce da bira klasteriranje sa ts podjednakih duljina

### why not cluster ts directly?
Jedini razlog kojeg sam se mogao sjetiti jest to da churn ne uzimam u obzir. Eventualno se moze odsjeci zadnjih N mjeseci prije churna.

Ili zato sto ne mozemo ukljuciti previse varijabli onda, preveliki noise i previse komputacije.

### biggest issue: mismatch in performance criteria
Najveci problem je po meni ovaj. Onako kako mi je to Kreso predlozio, ja radim a) staticku segmentaciju kako bi napravio dobru b) ts segmentaciju, a sve to kako bi c) staticki nasao churn prediktore.

No moguce je da sam ja krivo konceptualizirao problem.

## finding churn predictors (symptoms)
Definirati churn kao prestanak kladjenja od minimalno X mjeseci (tipa 3). Uzeti N mjeseci prije churn-a i pretvoriti to u staticke varijable. Gledati koje varijable predvidjaju churn. 

 - da li razlicite definicije churn-a za razlicite segmente?

Prijedlozi varijabli:

 - postotak pobjeda
 - trend gubitaka (npr 2nd degree poly koeficijenti)
 - broj dana od zadnje pobjede
 - varijabilitet tipa oklada (koeficijenti, na sto se kladi)
 - totalno stanje racuna u tom periodu

## finding churn distal predictors (causes)
Jedino sto mi pada na pamet jest napraviti istu staticku analizu kao i ranije, samo ovaj put vise mjeseci unaprijed.

# main questions

clustering on time series or static info?

what about survival analysis?

using static variables to predict churn (e.g., characteristics of a 3 month window to predict churn probability next month). It seems to me that this is the typical procedure.
 - http://research.ijcaonline.org/volume42/number20/pxc3878122.pdf
 - http://blog.yhat.com/posts/predicting-customer-churn-with-sklearn.html
 - Kreso kaze da je to "poor man's churn model", kad nemas dovoljno vremena

multidimensional (e.g., frequency and average bet size) or univariate?

which distance metric?
 - Euclidean, Manhattan
 - Dynamic Time Warping
 - characteristics based
     + but what if you need to transform each ts manually?

how to deal with unequal lengths?
 - cut them
 - use only sequences of length N

# definiranje konzistentnosti vremenskih serija
Prva ideja:
 - odbaciti sve podatke gdje ljudi jos nisu poceli sa igrom
 - normalizirati svaku vremensku seriju na M = 0, SD = 1. Ovako odbacujem razlike u velicini oklada. Ne znam jos da li je to dobro, ali segmenti ce se djelomicno temeljiti na velicini oklade tako da ce biti manja varijanca
 - izracunati prosjecnu vremensku seriju, a onda i varijancu od nje
 - (bez detalja zasad) izracunati nekakvu prosjecnu varijancu po periodu koja ima smisla
 - usporediti tu varijancu. Opcije: usporedjivati iste vremenske periode (= ocekujem isti sezonalitet i reakcije na vanjske uvjete), poredati sve serije otpocetka nadalje (= ocekujem isti lifecycle), odbaciti sezonalitet i poredati serije

A zasto ne klasterirati odmah na vremenskim serijama?
 - zato sto ce oni koji su odustali upasti u istu seriju?

## options

Dynamic time warping
 - radi na nejednakim velicinama: http://stats.stackexchange.com/questions/58725/time-series-similarity-differing-lengths-with-r?rq=1
 - http://dtw.r-forge.r-project.org/
 - google search
 - http://alexminnaar.com/time-series-classification-and-clustering-with-python.html
 - in R: http://www.rdatamining.com/examples/time-series-clustering-classification

http://www.ncbi.nlm.nih.gov/pmc/articles/PMC3564859/
http://journals.plos.org/plosone/article?id=10.1371/journal.pone.0054201
 - distance measures: Euclidean distance, the Manhattan distance (taxicab distance), Dynamic Time Warping (DTW), the Longest Common Subsequence (LCSS).
 - this paper proposes SMETS which is for multivariate time series

learned pattern similarity
 - https://cran.r-project.org/web/packages/LPStimeSeries/index.html
 - http://www.mustafabaydogan.com/learned-pattern-similarity-lps.html

## compressing?
You can compress time series to reduce the time it takes to compare them. (http://www.ncbi.nlm.nih.gov/pmc/articles/PMC3564859/pdf/pone.0054201.pdf)

# asked Davor

klasterirat vremenske nizove prema kros korelaciji. Minimal spanning tree dendrograms

# clustering time series directly
Ako ovim putem, moglo bi se desiti da svi churnevi odu u jedan kos.

can't use typical cluster analysis as the variables are correlated?

https://www.researchgate.net/publication/230800046_Clustering_Time_Series_of_Different_Length_Using_Self-Organising_Maps

## posts
http://stats.stackexchange.com/questions/3331/is-it-possible-to-do-time-series-clustering-based-on-curve-shape

http://robjhyndman.com/hyndsight/tscharacteristics/

http://pascal.iseg.utl.pt/~jcaiado/Papers/Paper_LSSP0213_Revised.pdf

http://stats.stackexchange.com/questions/2777/modelling-longitudinal-data-where-the-effect-of-time-varies-in-functional-form-be

http://stats.stackexchange.com/questions/3238/time-series-clustering-in-r

using kNN
 - http://alexminnaar.com/time-series-classification-and-clustering-with-python.html

http://www.cs.cmu.edu/~badityap/papers/clds-icml11.pdf


## packages / methods
https://cran.r-project.org/web/packages/kml/index.html

https://cran.r-project.org/web/packages/TSclust/TSclust.pdf

http://www.cs.ucr.edu/~eamonn/SAX.htm

https://cran.r-project.org/web/packages/pdc/pdc.pdf

http://www.cs.columbia.edu/~gravano/Papers/2015/sigmod2015.pdf

KNN: http://alexminnaar.com/time-series-classification-and-clustering-with-python.html

characteristics based (global characteristics):
 - http://www.robjhyndman.com/papers/DMKD.pdf

## issue: nejednake duljine i start points

# notes
automatic selection of ARIMA models
http://stats.stackexchange.com/questions/32742/auto-arima-vs-autobox-do-they-differ