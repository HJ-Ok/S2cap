# S2cap ♥: A Benchmark and a Baseline for Singing Style Captioning


## Detailed information beyond paper
<br>

### Detailed dataset information
<a id="detail-data-information"></a>

In this section, we provide more detailed statics of our dataset. Figure 1 (a) visualizes the top 20 most frequent Last.fm tags. In particular, we observe that `Korean` and `K-pop` are at the top. This trend arises because the Melon Playlist Dataset originates from Melon, a South Korean music streaming service, leading to a significant proportion of Korean music in the dataset.

Figure 1 (b), (c), and (d) illustrate the age distribution of the singer, the genre distribution of the song, and the duration distribution of the audio, respectively. The data reveal a broad coverage of various age groups and musical genres, highlighting the diversity of the data set. Regarding audio durations, we observe a specific distribution pattern, which results from our preprocessing step of segmenting audio clips into 5--30 seconds segments. This controlled segmentation ensures consistency while preserving essential musical features for downstream tasks.

<p align="center">
  <img src="/figures/S2cap_stats_detail_2.png" width="800"/>
  <br>
  <b>Figure 1:</b> Visualization of more statistical information for S2Cap.
</p>


### Data generation process example
<a id="generation-example"></a>

In this section, we provide a detailed explanation with an example process: A detailed process of web scraping and acoustic information extractor process.
Figure 2 illustrates the detailed web scraping process. Specifically, we used the BeautifulSoup library to extract tag information from Last.fm and retrieve artist metadata such as birth year and activity type from Melon. The corresponding URLs used for data extraction are depicted in Figure 2. For YouTube to download the WAV file, we employed Selenium and [youtube-dl](https://github.com/ytdl-org/youtube-dl) to conduct search queries and download the WAV file as illustrated in the figure. We extracted the top 20 search results and selected the audio file with the most similar duration to the original track.

After collecting the data through web scraping, we applied a three-stage filtering process to ensure data quality, as shown in Figure 2. The filtering criteria were as follows:

* Was the singer born before 1970?
* Is the Last.fm tag information missing?
* Does the duration of the downloaded WAV file significantly differ from the length metadata in the Melon playlist?

The corresponding data entry was filtered if any of these conditions were met. In particular, for duration differences, we filtered out instances where the difference between the extracted audio file and the metadata-reported length exceeded 1%.

<p align="center">
  <img src="/figures/S2cap_detailed_process.png" width="650"/>
  <br>
  <b>Figure 2:</b> Detailed data collecting example.
</p>

After obtaining WAV files from YouTube during the web scraping process, we applied an acoustic information extraction pipeline, as illustrated in Figure 3. First, we processed the wav files using HT-Demucs, a demixing model, to isolate the vocal tracks. We then applied a speaker diarization model to identify the speech segments of speakers. Using the segments of the most prevalent speaker, we segmented both the vocal WAV file and the original WAV file identically. The segmented portions of the original WAV file were then utilized as inputs for the S2Cap dataset.

Next, we extracted acoustic information from each segment. The original wave file segments were processed using the Qwen-2 Audio model to extract mood and tempo attributes and descriptions of the singing style. The demixed WAV file was also analyzed with the Qwen-2 Audio model to obtain timbre and gender attributes. Additionally, we employed an SLU model to extract pitch and RMS values. Based on these pitch and RMS values, we applied Algorithm 1 to derive the final pitch and volume information.

<p align="center">
  <img src="/figures/S2cap_detailed_process_2.png" width="800"/>
  <br>
  <b>Figure 3:</b> Detailed Acoustic information extractor example.
</p>

### Algorithm 1: Mapping Pitch and Volume Categories

#### Step 1: Categorize Pitch (per Gender)

1. For each `gender` in {**male**, **female**}:
    - Compute the 33% and 66% quantiles (`q33`, `q66`) of `pitch_value` for the current gender.
    - For each row `r` in `df` where `r.Gender == gender`:
        - If `r.pitch_value ≤ q33`, then set `r.Pitch = 'low'`.
        - Else if `r.pitch_value ≤ q66`, then set `r.Pitch = 'normal'`.
        - Else, set `r.Pitch = 'high'`.

#### Step 2: Categorize Volume (globally)

2. Compute the 33% and 66% quantiles (`v33`, `v66`) of `rms_value` across **all data**.
3. For each row `r` in `df`:
    - If `r.rms_value ≤ v33`, then set `r.Volume = 'low'`.
    - Else if `r.rms_value ≤ v66`, then set `r.Volume = 'normal'`.
    - Else, set `r.Volume = 'high'`.



### Prompts
<a id="generation-prompts"></a>

This section describes the prompt for generating a dataset utilizing Qwen-2 Audio and GPT-4o-2024-08-06. The prompt for Qwen-2 Audio is detailed below, followed by the prompt for GPT-4o. These prompts are inspired by G-Eval.

---

#### **Prompt for Singing Vocal Attributes**

You will be given information about specific singing vocal. Your task is to get attributes from the given audio.

Please make sure you read and understand these instructions carefully. Please keep this document open while reviewing, and refer to it as needed.

**Generation Steps**
1.  Listen to the specific audio and judge the following attributes: Gender, Pitch, Volume, Tempo, Mood, Timbre, Singing style description.
2.  Classify Gender, Pitch, Volume, and Tempo using the following options:
    * Gender: [`male`, `female`]
    * Pitch: [`low`, `normal`, `high`]
    * Tempo: [`slow`, `normal`, `fast`]
    * Volume: [`low`, `normal`, `high`]
3.  Provide detailed descriptions for Mood, Timbre, and Singing style.

**Example**

**Generation Form:**
  * Gender: male
  * Pitch: low
  * Tempo: slow
  * Volume: normal
  * Mood: sad
  * Timbre: warm and resonant
  * Singing style description: Ballad intro part with a low, soft, and trembling tone that is deep and full of yearning.



---

#### **Prompt for Singing Style Generation**
You will be given information about specific singing vocals. Your task is to get a singing style prompt in English from the given singing audio information. Please read and understand these instructions carefully. Keep this document open while reviewing and refer to it as needed.

**Generation Criteria**
1.  Make a singing style prompt in English based on the given source text.
2.  The prompt should not contain the artist's name, and not all elements should be reflected in the prompt.
3.  Gender, pitch, tempo, volume, mood, and timbre must be included.
4.  The prompt must be in one sentence.

**Generation Steps**
1.  Read the reference for specific singing audio about the following attributes: Artist name, Genre, Tags, Gender, Pitch, Volume, Tempo, Mood, Timbre, Age, and Singing style description.
2.  Based on this given information, generate a singing style caption as a prompt that ends with one sentence that reflects the singer's singing style in the given singing audio.
3.  Do not include the artist's name in the prompt.

**Example**

**Source Text:**
  * Artist name: Kim Dong-ryul
  * Genre: ballad
  * Tags: korean, ballad, k-pop, singer-songwriter
  * Gender: male
  * Pitch: low
  * Tempo: slow
  * Volume: normal
  * Mood: happy
  * Timbre: warm and rich
  * Age: 29
  * Singing style description: The low notes in the melody are sung with warmth and depth, conveying happiness and comfort.

**Generation Form:**
A gentle, low-pitched male vocal with warm and rich timbre delivering a slow, heartfelt ballad performance, expressing happiness and comfort through deep, resonant tones.


---

### Human evaluation guidelines
<a id="human-eval-guidelines"></a>

To compare the generated outputs with various models, we conducted a human evaluation with graduate students in which participants listened to an audio sample with two model-generated captions. They were then asked to determine which caption was superior or whether both were of similar quality (tie).

To assess the quality of captions generated by GPT-4o, we specifically evaluated consistency and fluency based on predefined criteria. Participants were first instructed to review the guidelines outlined in the **Prompt for caption evaluation: Consistency** and **Prompt for caption evaluation: Fluency** sections before listening to the audio and independently rating the captions for these two aspects.

In the objectivity evaluation, we provided participants with the singer's identity and asked whether it matched the expected timbre. We presented both an appropriate and randomly selected mismatched timbre, asking participants to identify whether the given voice was suitable.

---

#### **Prompt for caption evaluation: Consistency**
You will be given music audio and an audio caption describing the singing style of the singer in the music. Your task is to rate the audio caption using one metric. Please make sure you read and understand these instructions carefully.

**Evaluation Criteria**
* **Consistency (1-5)** - The factual alignment between the audio caption and the music audio. Annotators should penalize audio captions that contain hallucinated facts.

**Evaluation Steps**
1.  Listen to the audio carefully and identify the main facts and details they present in the singing style aspect.
2.  Read the singing style captions. Check if the audio caption contains any factual errors that are not supported by the music audio.
3.  Assign a score for Consistency based on the Evaluation Criteria.

---

#### **Prompt for caption evaluation: Fluency**
You will then be given one audio caption. Your task is to rate the audio caption using one metric. Please make sure you read and understand these instructions carefully.

**Evaluation Criteria**
* **Fluency (1-3)** - The quality of the audio caption in terms of grammar, spelling, punctuation, word choice, and sentence structure.
    * 1: Poor. The audio caption has many errors that make it hard to understand or sound unnatural.
    * 2: Fair. The audio caption has some errors that affect the clarity or smoothness of the text, but the main points are still comprehensible.
    * 3: Good. The audio caption has few or no errors and is easy to read and follow.
