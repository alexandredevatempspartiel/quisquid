# Quisquid

Speaker-attributed transcription of interviews, on Google Colab.

Transcription et attribution des locuteurs pour des entretiens, sur Google Colab.

[![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/Alexandredevatempspartiel/quisquid/blob/main/quisquid.ipynb)

[English](#english) · [Français](#français)

## English

Quisquid is a Google Colab notebook that transcribes audio or video recordings and indicates which speaker says each passage. It was first written for personal use, then made available to students at Sciences Po Bordeaux for transcribing interviews. It works with every language that Whisper recognises.

Diarisation indicates who is speaking and transcription indicates what is said.

### Features

- Transcription with Whisper large-v3, or with Whisper turbo for faster processing.
- Speaker attribution for up to eight voices, with a fixed number of speakers or automatic detection.
- Automatic language detection, a fixed language, or a multilingual mode for recordings that mix several languages.
- Optional context and vocabulary (proper names, acronyms, technical terms) to improve the recognition of rare words.
- Processing of a single file or of a batch of files with the same configuration.
- Export as a timestamped text file (.txt) and as a spreadsheet (.csv), with an optional copy in Google Drive.
- Interface in English or in French, chosen at startup.

### Usage

1. Open the notebook with the "Open in Colab" button.
2. Select the T4 GPU in Runtime > Change runtime type.
3. Run the first code cell, choose the interface language, then answer the configuration questions displayed below the cell.
4. Upload one or several files, then download the results with the buttons that appear at the end of the processing.

The first run installs the dependencies, which takes two to four minutes. The notebook accepts any format that ffmpeg can read, including video files.

### Output files

The text file gives, for each speaker turn, the start and end time, the speaker number and the spoken text. The CSV file contains the same information in columns (start_time, end_time, duration, speaker, text), so that it can be opened in a spreadsheet or processed with R or Python. Speakers are numbered in the order in which they first speak, and their labels follow the interface language.

### Technical design

The processing of each file consists of four steps.

1. Language detection. The Silero voice activity detector removes silence and noise, then Whisper evaluates the language on up to five 30-second windows distributed over the whole recording. In single-file mode, the notebook asks for confirmation when the detection confidence is below 70%.
2. Transcription. faster-whisper produces a timestamp for each word. Conditioning on the previous text is disabled to prevent a recognition error from repeating in the following windows or from producing repetition loops. The terms supplied by the user are inserted again into every 30-second window.
3. Diarisation. The NVIDIA Nemotron 3 Diarization model computes an activity probability for each of the eight possible speakers every 10 milliseconds. The offline mode of Transformers processes the recording in blocks of 30.4 seconds connected by a speaker cache, which keeps the identity of each voice consistent over the whole recording. Channels that are active for less than one second in total are discarded.
4. Word attribution. Each word receives the speaker who is most active during its duration. A word located outside every detected speech region receives the speaker of the nearest active region, so that no word is removed. The words are then grouped into sentences, at each final punctuation mark or after a pause of more than 1.5 seconds. Within a sentence, a speaker change that lasts less than 0.6 seconds is reassigned to the neighbouring speaker, because it usually results from imprecise word timestamps.

CTranslate2, the computation engine of faster-whisper, loads the CUDA 12 library libcublas.so.12, whereas Colab provides CUDA 13. The notebook installs this library with pip and loads it into memory before the first computation, which avoids modifying LD_LIBRARY_PATH.

If diarisation fails on the GPU, the notebook runs it again on the CPU. If it fails completely, the transcription is still saved, without speaker distinction.

All interface messages are stored in a single dictionary with a French and an English version of each message, so that a new language only requires a new entry for each message.

### Limitations

- The diarisation model distinguishes at most eight speakers.
- When several people speak at the same time, each word is attributed to a single speaker.
- Without a GPU, transcription is very slow, as expected.

### Privacy

Files are processed on the Colab virtual machine assigned to the user, which Google hosts. The notebook does not send the audio to any other service. It deletes the uploaded file at the end of its processing, and the transcriptions remain on the virtual machine until the end of the session. The models are downloaded from Hugging Face. For interviews that contain personal data, each user must check that this way of processing complies with the rules of their institution and of their research protocol.

### Licence

This project is distributed under the PolyForm Noncommercial License 1.0.0, whose text is in the file [LICENSE.md](LICENSE.md). It permits use, modification and redistribution for any noncommercial purpose, in particular by students, researchers and educational institutions. Any copy, even partial, must keep the "Required Notice" line that identifies the author. Commercial use requires a separate licence granted by the author.

Required Notice: Copyright (c) 2026 Alexandre Mahieu

### Third-party components

The notebook uses faster-whisper, Transformers, the Whisper models by OpenAI and the Nemotron 3 Diarization model by NVIDIA. These components are downloaded at run time and remain subject to their own licences. The interface icons come from Lucide (ISC licence) and are derived from the Feather project (MIT licence). The full text of these two licences is at the end of the notebook.

## Français

Quisquid est un notebook Google Colab qui transcrit des enregistrements audio ou vidéo et indique quel locuteur prononce chaque passage. Il a d'abord été écrit pour un usage personnel, puis mis à disposition des étudiants de Sciences Po Bordeaux pour transcrire des entretiens. Il fonctionne avec toutes les langues reconnues par Whisper.

La diarisation indique qui parle et la transcription indique ce qui est dit.

### Fonctionnalités

- Transcription avec Whisper large-v3, ou avec Whisper turbo pour un traitement plus rapide.
- Attribution des locuteurs jusqu'à huit voix, avec un nombre de locuteurs imposé ou détecté automatiquement.
- Détection automatique de la langue, langue imposée, ou mode multilingue pour les enregistrements qui mélangent plusieurs langues.
- Contexte et vocabulaire optionnels (noms propres, sigles, termes techniques) pour améliorer la reconnaissance des mots rares.
- Traitement d'un fichier unique ou d'un lot de fichiers avec la même configuration.
- Export en texte horodaté (.txt) et en tableur (.csv), avec copie optionnelle dans Google Drive.
- Interface en anglais ou en français, au choix au démarrage.

### Utilisation

1. Ouvrir le notebook avec le bouton « Open in Colab ».
2. Sélectionner le GPU T4 dans Exécution > Modifier le type d'exécution.
3. Lancer la première cellule de code, choisir la langue de l'interface, puis répondre aux questions de configuration affichées sous la cellule.
4. Importer le ou les fichiers, puis télécharger les résultats avec les boutons qui apparaissent à la fin du traitement.

La première exécution installe les dépendances, ce qui prend deux à quatre minutes. Le notebook accepte tout format lisible par ffmpeg, y compris les fichiers vidéo.

### Fichiers produits

Le fichier texte indique, pour chaque intervention, l'heure de début et de fin, le numéro du locuteur et le texte prononcé. Le fichier CSV contient les mêmes informations en colonnes (start_time, end_time, duration, speaker, text), ce qui permet de l'ouvrir dans un tableur ou de le traiter avec R ou Python. Les locuteurs sont numérotés dans l'ordre de leur première prise de parole, et leur libellé suit la langue de l'interface.

### Fonctionnement technique

Le traitement de chaque fichier comporte quatre étapes.

1. Détection de la langue. Le détecteur d'activité vocale Silero retire les silences et le bruit, puis Whisper évalue la langue sur cinq fenêtres de 30 secondes au maximum, réparties sur l'ensemble de l'enregistrement. En mode fichier unique, le notebook demande une confirmation quand la confiance de la détection est inférieure à 70 %.
2. Transcription. faster-whisper produit un horodatage pour chaque mot. Le conditionnement sur le texte précédent est désactivé pour éviter qu'une erreur de reconnaissance se répète dans les fenêtres suivantes ou produise des boucles de répétition. Les termes fournis par l'utilisateur sont réinjectés dans chaque fenêtre de 30 secondes.
3. Diarisation. Le modèle NVIDIA Nemotron 3 Diarization calcule toutes les 10 millisecondes une probabilité d'activité pour chacun des huit locuteurs possibles. Le mode hors ligne de Transformers traite l'enregistrement par blocs de 30,4 secondes reliés par un cache de locuteurs, ce qui conserve l'identité de chaque voix sur toute la durée. Les canaux actifs moins d'une seconde au total sont écartés.
4. Attribution des mots. Chaque mot reçoit le locuteur le plus actif pendant sa durée. Un mot situé hors de toute zone de parole détectée reçoit le locuteur de la zone active la plus proche, de sorte qu'aucun mot n'est supprimé. Les mots sont ensuite regroupés en phrases, à chaque ponctuation finale ou après une pause de plus de 1,5 seconde. À l'intérieur d'une phrase, un changement de locuteur qui dure moins de 0,6 seconde est rattaché au locuteur voisin, parce qu'il provient le plus souvent de l'imprécision des horodatages de mots.

CTranslate2, le moteur de calcul de faster-whisper, charge la bibliothèque libcublas.so.12 de CUDA 12, alors que Colab fournit CUDA 13. Le notebook installe cette bibliothèque avec pip et la charge en mémoire avant le premier calcul, ce qui évite de modifier LD_LIBRARY_PATH.

Si la diarisation échoue sur le GPU, le notebook la relance sur le processeur. Si elle échoue complètement, la transcription est tout de même enregistrée, sans distinction de locuteurs.

Tous les messages de l'interface sont réunis dans un dictionnaire unique qui contient une version française et une version anglaise de chaque message. L'ajout d'une langue demande seulement une nouvelle entrée pour chaque message.

### Limites

- Le modèle de diarisation distingue au maximum huit locuteurs.
- Quand plusieurs personnes parlent en même temps, chaque mot est attribué à un seul locuteur.
- Sans GPU, la transcription est très lente, comme attendu.

### Confidentialité

Les fichiers sont traités sur la machine virtuelle Colab attribuée à l'utilisateur, hébergée par Google. Le notebook n'envoie l'audio à aucun autre service. Il supprime le fichier importé à la fin de son traitement, et les transcriptions restent sur la machine virtuelle jusqu'à la fin de la session. Les modèles sont téléchargés depuis Hugging Face. Pour des entretiens qui contiennent des données personnelles, chaque utilisateur doit vérifier que ce mode de traitement respecte les règles de son établissement et de son protocole de recherche.

### Licence

Ce projet est distribué sous la licence PolyForm Noncommercial 1.0.0, dont le texte figure dans le fichier [LICENSE.md](LICENSE.md). Elle autorise l'utilisation, la modification et la redistribution à toute fin non commerciale, notamment par les étudiants, les chercheurs et les établissements d'enseignement. Toute copie, même partielle, doit conserver la mention « Required Notice » qui désigne l'auteur. Un usage commercial nécessite une licence distincte accordée par l'auteur.

Required Notice: Copyright (c) 2026 Alexandre Mahieu

### Composants tiers

Le notebook utilise faster-whisper, Transformers, les modèles Whisper d'OpenAI et le modèle Nemotron 3 Diarization de NVIDIA. Ces composants sont téléchargés au moment de l'exécution et restent soumis à leurs propres licences. Les icônes de l'interface proviennent de Lucide (licence ISC) et dérivent du projet Feather (licence MIT). Le texte complet de ces deux licences figure à la fin du notebook.
