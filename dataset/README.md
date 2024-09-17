## Dataset
### S2Cap
You can download our dataset [here](https://huggingface.co/datasets/HJOK/S2Cap)

- **id**: A unique identifier from the **Melon Playlist Dataset**.
- **caption**: Generated captions describing the singing style of the track.
- **begin**: The start time (in seconds) of the audio segment in the `youtube_id` video.
- **end**: The end time (in seconds) of the audio segment in the `youtube_id` video.
- **youtube_id**: The YouTube ID of the video containing the audio track used.


### Data generate pipeline
![S2caps_datapipline](https://github.com/user-attachments/assets/da101ac3-2bd8-4cc3-bf4c-5d8dc3652a5b)
This dataset uses [**Melon Playlist Dataset**](https://mtg.github.io/melon-playlist-dataset/) and includes training, development, and test sets.

### Statistics
| Train  | Dev   | Test   | Total  |
|--------|-------|--------|--------|
| 48,825 | 7,509 | 14,881 | 71,215 |
