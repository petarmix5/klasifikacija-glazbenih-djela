# klasifikacija-glazbenih-djela
Music classification - analiza glazbenih signala

Ovaj projekt je napravljen u sklopu kolegija "Osnove digitalne obrade govora i slika" na Odjelu za informatiku, Sveučilišta u Rijeci.


<pre>
<code>


import numpy as np
from matplotlib import pyplot as plt 
from IPython.display import Audio, display
#import ipywidgets as widgets

plt.rcParams['figure.figsize'] = [10, 5]

def funkcija(trajanje, frekvencija, frekvencijaUzorkovanja, fazniPomak, amplituda):
  #uzorci su od 0 do postavljene vrijednosti u varijabli trajanje u sekundama
  vremenskaOs = np.arange(0, trajanje, 1/frekvencijaUzorkovanja)
  #izlazni rezutat će nam biti sinusoida zadanog trajanja frekvencije i faznog pomaka
  return amplituda * np.sin(2 * np.pi *frekvencija * vremenskaOs + fazniPomak)

#također se može generirati 1.5 sekunda sinusoide željene amplitude i frekvencije, 440Hz
fu =16000
tra = 1.5
f = 440
A = 0.4
fi = 0
x = funkcija(tra, f, fu, fi, A)

"""Ako je frekvencija uzorkovanja 16000, a trajanje 1.5 sekundu, tada će signal imati 24000 uzoraka"""

print(len(x))

"""Grafički prikaz signala te preslušavanje istog."""

plt.plot(x)
plt.xlabel("Uzorak signala")
plt.ylim(-1, 1)
Audio(x, rate=fu)

"""Za prikaz prvih nekoliko uzoraka koristimo plot naredbu.

Na ovom prikazu je vidljiva sinusoida koja se odnosi na vremenski kontinuirani signal.
"""

plt.plot(x[0:200])
plt.xlabel("Uzorak signala")
plt.ylim(-1, 1)

"""S obzirom na to da se signal sastoji od diskretnih uzoraka, može se prikazati z vremenski diskretni signal."""

plt.stem(x[0:200], use_line_collection=True)
plt.xlabel("Uzorak signala")
plt.ylim(-1, 1)

"""Da bi se vremenska os prikazala u sekundama, potrebno je zadati vremenske trenutke, a to se radi na sljedeći način."""

#prethodno napravljenu formulu kopiramo te napravimo neke izmjene 
def funkcija(trajanje, frekvencija, frekvencijaUzorkovanja, fazniPomak, amplituda):
  #uzorci su od 0 do postavljene vrijednosti u varijabli trajanje u sekundama
  vremenskaOs = np.arange(0, trajanje, 1/frekvencijaUzorkovanja)
  #izlazni rezutat će nam biti sinusoida zadanog trajanja frekvencije i faznog pomaka
  x = amplituda * np.sin(2 * np.pi *frekvencija * vremenskaOs + fazniPomak)
  return x, vremenskaOs

"""Zatim opet generiramo signal da bi dobili prikaz u sekundama. 
Vidljivo je da se na osi x više ne nalaze vrijednosti uzoraka već vrijeme[s]
"""

x, vremenskaOs =funkcija(tra, f, fu, fi, A)
plt.plot(vremenskaOs[0:200],x[0:200])
plt.xlabel("Vrijeme [s]")
plt.ylim(-1, 1)

"""# **Analiza glazbenih signala**"""

import IPython.display as ipd
zvuk = '/content/Hamburg.mp3'
ipd.Audio(zvuk)

!pip install essentia
import essentia
import essentia.standard as es

features, features_frames = es.MusicExtractor(lowlevelStats=['mean', 'stdev'],
                                              rhythmStats=['mean', 'stdev'], 
                                              tonalStats=['mean', 'stdev'])(zvuk)

"""Ispis imena svih značajki sortiranim redoslijedom"""

print(sorted(features.descriptorNames()))

"""Može se pristupiti svakoj vrijednosti posebno. Ispisuje se naziv datoteke, frekvencija uzorkovanja, ritam i druge značajke."""

print("Naziv datoteke: ", features['metadata.tags.file_name'])
print("-"*80)
print("Broj uzoraka: ", features['metadata.audio_properties.sample_rate'])
print("Ritam BPM(beats per min): ", features['rhythm.bpm'])
print("-"*80)
print("Glasnoća ritmičnog udarca: ", features['rhythm.beats_loudness.mean'])
print("Pozicije udarca u sekundama: ", features['rhythm.beats_position'])
print("-"*80)
print("Procjena ključa/skale (ljestvice): ", features['tonal.key_edma.key'], features['tonal.key_edma.scale'])

"""Sljedeća stvar koju radimo je detekcija otkucaja i BPM histogram tjst procjena tempa"""

from essentia.standard import *

audio = MonoLoader(filename="/content/Hamburg.mp3")()
rhythm_extractor = RhythmExtractor2013(method="multifeature")
bpm, beats, beats_confidence, _, beats_intervals = rhythm_extractor(audio)

print("BPM: ", bpm)
print("Pozicije udarca u sekundama: ", beats)
print("Beat estimation confidence: ", beats_confidence)

#Označavanje pozije udarca na audio zapisu te ih zapisujemo na datoteku 

marker = AudioOnsetsMarker(onsets=beats, type='beep')
marked_audio = marker(audio)
MonoWriter(filename='/content/Hamburg_s_udarcima.mp3')(marked_audio)

"""Možemo preslušati rezultat s udarcem koji je označen po sekundama okomitom crtom. Također ga možemo i vizualizirati """

import IPython 
IPython.display.Audio('/content/Hamburg_s_udarcima.mp3')

# Commented out IPython magic to ensure Python compatibility.
from pylab import plot, show, figure, imshow
# %matplotlib inline
import matplotlib.pyplot as plt
plt.rcParams['figure.figsize'] = (15, 6) #zadavanje veličine koja nije defaultna 

plot(audio)
for beat in beats: 
  plt.axvline(x=beat*44100, color='red')

plt.title("valni oblik zvuka i procjenjena pozicija klika/udarca")

"""Izlazna vrijednost BPM-a koju izdaje RhythmExtractor2013 je prosjek svih BPM-a za svaki interval između dva uzastopna otkucaja. """

peak1_bpm, peak1_weight, peak1_spread, peak2_bpm, peak2_weight, peak2_spread, histogram = BpmHistogramDescriptors()(beats_intervals)

print("Ukupni BPM (overall BPM): %0.1f" % bpm)
print("Prvi vrh histograma: %0.1f bpm)" % peak1_bpm)
print("Drugi vrh histograma: %0.1f bpm" % peak2_bpm)

fig, ax = plt.subplots()
ax.bar(range(len(histogram)), histogram, width=1)
ax.set_xlabel('BPM')
ax.set_ylabel('Frekvencija')
plt.title("BPM histogram")
ax.set_xticks([20 * x + 0.5 for x in range(int(len(histogram)/ 20))])
ax.set_xticklabels([str(20 * x) for x in range(int(len(histogram) / 20))])
plt.show()

"""Onset detection

Otkrivanje početka i označavanje nastanka na zvuku pomoću algoritma AudioOnsetsMarker.

Sastoji se od dvije faze: potrebno je izračunati funkciju otkrivanja početka koja je funkcija koja opisuje evoluciju parametara koji mogu biti reprezentativni za to hoćemo li pronaći početak ili ne. Odabrati mjesta početka u signalu na temelju broja funkcija otkrivanja. 

OnsetDetection algoritam procjenjuje različite funkcije za početak otkrivanja s obzirom na njegov spektar. Također je moguće izvorni zvuk i zvučne signale pohraniti u stereo signal stavljajući ih zasebno u lijevi i desni kanal pomoću StereoMuxera i AudioWritera.
"""

#Faza 1: izračunati funkciju za početak otkrivanja

onsetDetection1 = OnsetDetection(method = 'hfc')
onsetDetection2 = OnsetDetection(method= 'complex')

#dohvaćanje potrebnih algoritama i bazena (pool) za pohranu rezultata
w = Windowing(type = 'hann')
fft = FFT() #daje nam kompleksni FFT (fast Fourier Transform)

c2p = CartesianToPolar() #pretvara u par (magnituda, faza)
pool = essentia.Pool()

#Funkcija otkrivanja početka
for frame in FrameGenerator(audio, frameSize= 1024, hopSize= 512):
  magnituda, faza = c2p(fft(w(frame)))
  pool.add('features.hfc', onsetDetection1(magnituda, faza))
  pool.add('features.complex', onsetDetection2(magnituda, faza))

#Faza 2 Izračunati stvarna mjesta početka
onsets = Onsets()

onsets_hfc = onsets(# ovaj algoritam zahtjeva matricu, a ne vektor
                    essentia.array([pool['features.hfc']]),
                    #potrebno je specificirati težinu
                    #nije bitno koju veličinu zadajemo 
                    [1])

onsets_complex = onsets(essentia.array([pool['features.complex']]),[1])

#označi početak zvuka koji će se zapisati natrag na disk 
#koristimo zvučni signal umjesto prazne buke i stereo signal kao karakterističan

silence = [0.] * len(audio)

beeps_hfc = AudioOnsetsMarker(onsets=onsets_hfc, type='beep')(silence)
AudioWriter(filename='/content/Hamburg_onsets_hfc_stereo.mp3', format='mp3')(StereoMuxer()(audio, beeps_hfc))

beeps_complex = AudioOnsetsMarker(onsets=onsets_complex, type='beep')(silence)
AudioWriter(filename='/content/Hamburg_onsets_complex_stereo.mp3', format='mp3')(StereoMuxer()(audio, beeps_complex))

"""Pregled dobivenog rezultata zvučne datoteke i pregled koja funkcija radi bolje 

"""

IPython.display.Audio('/content/Hamburg_s_udarcima.mp3')

IPython.display.Audio('/content/Hamburg_onsets_hfc_stereo.mp3')

IPython.display.Audio('/content/Hamburg_onsets_complex_stereo.mp3')

"""Pregledavajući plotove s vertikalnim linijama koje označavaju početak, može se vidjeti kako je HFC metoda otkrila udarce dok je COMPLEX metoda isto otkrila udarce, ali se čuju i nakon završetka glazbe tj. dok je potpuna tišina. """

plot(audio)

"""HFC ONSET DETECTION FUNCTION"""

plot(audio)
for onset in onsets_hfc:
  plt.axvline(x=onset*44100, color = 'red')

plt.title("Valni oblik zvuka i procjenjena početna pozicija (HFC onset detection function")
plt.show()

"""COMPLEX ONSET DETECTION FUNCTION"""

plot(audio)
for onset in onsets_complex:
  plt.axvline(x=onset*44100, color='red')

plt.title("Valni oblik zvuka i procjenjena početna pozicija (COMPLEX onset detection function")

"""DETEKCIJA MELODIJE

Ovdje se analizira kontura visine (pitch conture) dominantne melodije u zvučnom zapisu pomoću algoritma PredominantPitchMelodia. Algoritam ispisuje slijed vrijednosti s trenutnom vrijednosti visine tona u Hz percipirane melodije.
"""

import numpy as np

print("Trajanje zvuka [sek]: ")
print(len(audio/44100.0))

#PitchMelodia uzima cijeli zvučni signal kao ulaz

pitch_extractor = PredominantPitchMelodia(frameSize=2048, hopSize=128)
pitch_values, pitch_confidence = pitch_extractor(audio)

#Procjenjeni korak na frameovima. Računanje pozicije framea

pitch_times = np.linspace(0.0, len(audio)/44100.0, len(pitch_values))

#Crtanje konture
f, axarr = plt.subplots(2, sharex = True)
axarr[0].plot(pitch_times, pitch_values)
axarr[0].set_title('estimated pitch [Hz]')
axarr[1].plot(pitch_times, pitch_confidence)
axarr[1].set_title('pitch confidence')
plt.show()

"""Preslušavanje procjenjene visine i usporedba s izvornim zvukom

IDENTIFIKACIJA OBRADE PJESME 

Cover song identification u MIR-u zadatak je identificiranja kada dvije glazbene snimke potječu iz iste skladbe. Primjer je SHAZAM. Obrada pjesme se može razlikovati od izvorne pjesme. 

Essentia nudi implementaciju otvorenog koda najsuvremenijih algoritama za identifikaciju obrade pjesama. Procesni lanac koji je potreban za korištenje CSI algoritama je: 
- izdvajanje tonskih značajki. Koriste se značajkama Chroma, a ovdje koristimo HPCP
- naknadna obrada značajki radi postizanja invarijantnosti (npr.Ključ)
- Izračun matrice unakrsne sličnosti 
- Lokalno poravnavanje podsljedova za izračunavanje udaljenosti sličnosti obrade pjesme u paru. 

Koristi se HPCP, CoverSongSimilarity te ChromaCrossSimilarity algoritmi iz biblioteke Essentia.
"""

import essentia.standard as estd 
from essentia.pytools.spectral import hpcpgram

"""Učitavamo pjesmu za obradu, referentnu pjesmu (pravu obradu) i referentnu pjesmu s lažnom obradom. 

Prva pjesma je upit za obradu pjesme i to je cover na pjesmu Hello od Adele.Druga pjesma je referentna pjesma odnosno prava obrada dok je treća pjesma lažna obrada metal.

Prva pjesma - upit za obradu pjesme - cover Davina Michelle
"""

IPython.display.Audio('/content/Hello - Adele (Cover By Davina Michelle).mp3')

"""Druga pjesma - original - Prava obrada """

IPython.display.Audio('/content/Adele-Hello.mp3')

"""Treća pjesma - referentna pjesma (Lažna obrada) - Reggae Cover """

IPython.display.Audio('/content/Hello - Adele (Reggae Cover) - Conkarah and Rosie Delmah.mp3')

"""Upit za obradu pjesme 

"""

upit_za_obradu = estd.MonoLoader(filename = '/content/Hello - Adele (Cover By Davina Michelle).mp3', sampleRate = 32000)()

prava_obrada_pjesme = estd.MonoLoader(filename = '/content/Adele-Hello.mp3', sampleRate = 32000)()

lazna_obrada_pjesme = estd.MonoLoader(filename = '/content/Hello - Adele (Reggae Cover) - Conkarah and Rosie Delmah.mp3', sampleRate=32000)()

"""Izračunavanje hromatskih značajki harmonijskih profila klase HPCP audio signala."""

upit_za_obradu_hpcp = hpcpgram(upit_za_obradu, sampleRate=32000)
prava_obrada_pjesme_hpcp = hpcpgram(prava_obrada_pjesme, sampleRate=32000)
lazna_obrada_pjesme_hpcp = hpcpgram(lazna_obrada_pjesme, sampleRate=32000)

"""Crtanje značajki hpcp-a"""

fig = plt.gcf()
fig.set_size_inches(14.5, 4.5)

plt.title("Upit za obradu HPCP")
plt.imshow(upit_za_obradu_hpcp[:500].T, aspect='auto', origin='lower', interpolation='none')

"""Pomoću funkcije ChromaCrossSimilarity se izvode sljedeći koraci: 
-slaganje ulaznih značajki
- ključna invarijantnost pomoću indeksa optimalne transpozicije (OTI)
- izračun sličnosti binarne unakrsne kroma pomoću unakrsnog ponavljajućeg grafikona ili korištenjem kroma binarne metode temeljene na OTI
"""

chroma_cross_Similarity = estd.ChromaCrossSimilarity(frameStackSize = 9,
                                                   frameStackStride = 1, 
                                                   binarizePercentile = 0.095,
                                                   oti = True)

pravi_par_crp = chroma_cross_Similarity(upit_za_obradu_hpcp, prava_obrada_pjesme_hpcp)

fig = plt.gcf()
fig.set_size_inches(15.5, 5.5)

plt.title("Dijagram unakrsnog ponavljanja ")
plt.xlabel("Adele - Hello cover od Michelle Davina")
plt.ylabel("Adele - Hello - original")
plt.imshow(pravi_par_crp, origin='lower')

"""Izračun sličnosti binarne unakrsne kroma pomoću unakrsnog ponavljajućeg grafikona parova koji ne pokrivaju """

chroma_cross_Similarity = estd.ChromaCrossSimilarity(frameStackSize = 9,
                                                     frameStackStride = 1,
                                                     binarizePercentile = 0.095,
                                                     oti = True)

lazni_par_crp = chroma_cross_Similarity(upit_za_obradu_hpcp, lazna_obrada_pjesme_hpcp) #usporedba sličnosti prve pjesme za upit i lažne obrade

fig = plt.gcf()
fig.set_size_inches(15.5, 5.5)

plt.title('Dijagram unakrsnog ponavljanja')
plt.xlabel("Adele - Reggae - cover")
plt.ylabel("Adele - Hello - original")
plt.imshow(lazni_par_crp, origin="lower")

"""Metoda binarne sličnosti na temelju OTI za izračun unakrsne sličnosti dviju zadanih značajki boje """

chroma_cross_Similarity = estd.ChromaCrossSimilarity(frameStackSize = 9, 
                                                     frameStackStride = 1, 
                                                     binarizePercentile = 0.095,
                                                     oti = True,
                                                     otiBinary = True)

oti_chroma_cross_Similarity = chroma_cross_Similarity(upit_za_obradu_hpcp, lazna_obrada_pjesme_hpcp)

fig = plt.gcf()
fig.set_size_inches(15.5, 5.5)

plt.title("Unakrsna sličnost matrica koristeći OTI binarnu metodu")
plt.xlabel("Adele - Reggae - cover")
plt.ylabel("Adele - Hello - original")
plt.imshow(oti_chroma_cross_Similarity, origin="lower")

"""Izračunavamo mjeru sličnosti asimetrične obrade pjesama iz unaprijed izračunate binarne matrice sličnosti omota pomoću različitih kontrasta algoritma poravnavanja redoslijeda Smith-Waterman (serra09 ili chen17)

Računanje udaljenosti sličnosti obrade pjesme između *Adele Hello Davina Michelle cover* i *Adele Hello original*.
"""

binarna_matrica, udaljenost = estd.CoverSongSimilarity(disOnset = 0.5,
                                                       disExtension = 0.5, 
                                                       alignmentType = 'serra09',
                                                       distanceType='asymmetric')(pravi_par_crp)

fig = plt.gcf()
fig.set_size_inches(15.5, 5.5)

plt.title("Udaljenost sličnosti obrade pjesme: %s" % udaljenost)
plt.xlabel("Adele - Hello - Davina Michelle cover")
plt.ylabel("Adele - Hello - original")
plt.imshow(binarna_matrica, origin='lower')

print("Udaljenost sličnosti obrade pjesme je: %s" % udaljenost)

"""Računanje sličnosti obrade pjesme udaljenosti između *Adele Reggae cover* i *Adele Hello original*"""

binarna_matrica, udaljenost = estd.CoverSongSimilarity(disOnset = 0.5,
                                                       disExtension = 0.5,
                                                       alignmentType = 'serra09',
                                                       distanceType ='asymmetric')(lazni_par_crp)

fig = plt.gcf()
fig.set_size_inches(15.5, 5.5)

plt.title("Udaljenost sličnosti obrade pjesme je: %s" % udaljenost)
plt.xlabel("Adele - Reggae - cover")
plt.ylabel("Adele - Hello - original")
plt.imshow(binarna_matrica, origin='lower')

print("Udaljenost sličnosti obrade pjesme je: %s" % udaljenost)

"""Iz prikaza iznad se može vidjeti da je udaljenost obrade prilično niska za stvarne parove obrada pjesama, a upravo se tako nizak rezultat mogao i očekivati.



-------------------------------------------------------------------------

#------------------------------------------------------------------------------------

Naknadno dodano iza obrane projekta (dodavanje treće pjesme i razlika Onset i BPM)

Dodali smo treću pjesmu koja nema veze s prethodnima niti bilo kakvu sličnost kako bi prikazali razliku odnosno udaljenost sličnosti obrade pjesama koja bi trebala biti znatno veća nego što je prikazano na prethodnim rezultatima. Treća pjesma koju smo dodali je Shakira - Chantaje.
"""

IPython.display.Audio('/content/Shakira - Chantaje (Official Video) ft. Maluma.mp3')

"""Upit za obradu pjesme 

"""

upit_za_obradu = estd.MonoLoader(filename = '/content/Hello - Adele (Cover By Davina Michelle).mp3', sampleRate = 32000)()

prava_obrada_pjesme = estd.MonoLoader(filename = '/content/Adele-Hello.mp3', sampleRate = 32000)()

lazna_obrada_pjesme = estd.MonoLoader(filename = '/content/Shakira - Chantaje (Official Video) ft. Maluma.mp3', sampleRate=32000)()

"""Izračunavanje hromatskih značajki harmonijskih profila klase HPCP audio signala."""

upit_za_obradu_hpcp = hpcpgram(upit_za_obradu, sampleRate=32000)
prava_obrada_pjesme_hpcp = hpcpgram(prava_obrada_pjesme, sampleRate=32000)
lazna_obrada_pjesme_hpcp = hpcpgram(lazna_obrada_pjesme, sampleRate=32000)

"""Crtanje značajki hpcp-a"""

fig = plt.gcf()
fig.set_size_inches(14.5, 4.5)

plt.title("Upit za obradu HPCP")
plt.imshow(upit_za_obradu_hpcp[:500].T, aspect='auto', origin='lower', interpolation='none')

"""Pomoću funkcije ChromaCrossSimilarity se izvode sljedeći koraci: 
-slaganje ulaznih značajki
- ključna invarijantnost pomoću indeksa optimalne transpozicije (OTI)
- izračun sličnosti binarne unakrsne kroma pomoću unakrsnog ponavljajućeg grafikona ili korištenjem kroma binarne metode temeljene na OTI
"""

chroma_cross_Similarity = estd.ChromaCrossSimilarity(frameStackSize = 9,
                                                   frameStackStride = 1, 
                                                   binarizePercentile = 0.095,
                                                   oti = True)

pravi_par_crp = chroma_cross_Similarity(upit_za_obradu_hpcp, prava_obrada_pjesme_hpcp)

fig = plt.gcf()
fig.set_size_inches(15.5, 5.5)

plt.title("Dijagram unakrsnog ponavljanja ")
plt.xlabel("Adele - Hello cover od Michelle Davina")
plt.ylabel("Adele - Hello - original")
plt.imshow(pravi_par_crp, origin='lower')

"""Izračun sličnosti binarne unakrsne kroma pomoću unakrsnog ponavljajućeg grafikona parova koji ne pokrivaju """

chroma_cross_Similarity = estd.ChromaCrossSimilarity(frameStackSize = 9,
                                                     frameStackStride = 1,
                                                     binarizePercentile = 0.095,
                                                     oti = True)

lazni_par_crp = chroma_cross_Similarity(upit_za_obradu_hpcp, lazna_obrada_pjesme_hpcp) #usporedba sličnosti prve pjesme za upit i lažne obrade

fig = plt.gcf()
fig.set_size_inches(15.5, 5.5)

plt.title('Dijagram unakrsnog ponavljanja')
plt.xlabel("Adele - Reggae - cover")
plt.ylabel("Shakira - Chantaje")
plt.imshow(lazni_par_crp, origin="lower")

"""Metoda binarne sličnosti na temelju OTI za izračun unakrsne sličnosti dviju zadanih značajki boje """

chroma_cross_Similarity = estd.ChromaCrossSimilarity(frameStackSize = 9, 
                                                     frameStackStride = 1, 
                                                     binarizePercentile = 0.095,
                                                     oti = True,
                                                     otiBinary = True)

oti_chroma_cross_Similarity = chroma_cross_Similarity(upit_za_obradu_hpcp, lazna_obrada_pjesme_hpcp)

fig = plt.gcf()
fig.set_size_inches(15.5, 5.5)

plt.title("Unakrsna sličnost matrica koristeći OTI binarnu metodu")
plt.xlabel("Adele - Reggae - cover")
plt.ylabel("Shakira - Chantaje")
plt.imshow(oti_chroma_cross_Similarity, origin="lower")

"""Izračunavamo mjeru sličnosti asimetrične obrade pjesama iz unaprijed izračunate binarne matrice sličnosti omota pomoću različitih kontrasta algoritma poravnavanja redoslijeda Smith-Waterman (serra09 ili chen17)

Računanje udaljenosti sličnosti obrade pjesme između *Adele Hello Davina Michelle cover* i *Adele Hello original*.
"""

binarna_matrica, udaljenost = estd.CoverSongSimilarity(disOnset = 0.5,
                                                       disExtension = 0.5, 
                                                       alignmentType = 'serra09',
                                                       distanceType='asymmetric')(pravi_par_crp)

fig = plt.gcf()
fig.set_size_inches(15.5, 5.5)

plt.title("Udaljenost sličnosti obrade pjesme: %s" % udaljenost)
plt.xlabel("Adele - Hello - Davina Michelle cover")
plt.ylabel("Adele - Hello - original")
plt.imshow(binarna_matrica, origin='lower')

print("Udaljenost sličnosti obrade pjesme je: %s" % udaljenost)

"""Računanje sličnosti obrade pjesme udaljenosti između *Adele Reggae cover* i *Shakira Chantaje*"""

binarna_matrica, udaljenost = estd.CoverSongSimilarity(disOnset = 0.5,
                                                       disExtension = 0.5,
                                                       alignmentType = 'serra09',
                                                       distanceType ='asymmetric')(lazni_par_crp)

fig = plt.gcf()
fig.set_size_inches(15.5, 5.5)

plt.title("Udaljenost sličnosti obrade pjesme je: %s" % udaljenost)
plt.xlabel("Adele - Reggae - cover")
plt.ylabel("Shakira - Chantaje")
plt.imshow(binarna_matrica, origin='lower')

print("Udaljenost sličnosti obrade pjesme je: %s" % udaljenost)

"""Vidljivo je na temelju prethodnog rezultata da je udaljenost znatno veća kao što se moglo očekivati zbog toga što pjesme nisu povezane ni na koji način.

ONSET algoritam izračunava početne položaje s obzirom na različite detekcije početka funkcije, dok BPM algoritam označava broj otkucaja u minuti. BPM se odnosi na broj otkucaja koji se trebaju otkucati u minuti kako bi se mjerio tempo pjesme.
"""
</code>
</pre>
