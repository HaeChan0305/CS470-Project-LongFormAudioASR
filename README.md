# Accurate long-form audio transcription via voice activity detection and forced alignment
This is the final project for Professor [Daehyung Park](https://sites.google.com/site/daehyungpark)'s course, **CS470: Introduction to Artificial Intelligence**, in **KAIST**.
Check our poster [poster](https://drive.google.com/file/d/1DkdCQA2iFCr-5cryAgEMVAc2QEJIi1Jh/view?usp=sharing).


## Problem Formulation
Progress in speech recognition has been boosted by the development of weakly supervised pre-training techniques exemplified by Whisper (Radford et al., 2022). Despite Whisper’s remarkable results in various experiments, there are still some chronic problems in the field of speech recognition.

### Problem
Most of speech recognition models including Whisper struggle with long-from audio transcription, stuck in a loop generating repetitive tokens or producing no token at all.

<p align="center">
  <img src="/Figure/table1.png" width="50%" alt="Table 1">
</p>


### Challenge
It is unclear why the phenomenon of falling into repeated loops occurs. In order to find out the most effective influential cause, analyzing each word one by one was inevitable to check when this phenomenon occurs.

### Solution
Since Whisper relies on timestamp tokens to determine the amount to shift the model’s context window by, factors hindering prediction on timestamp tokens (e.g. filler words, silence) may negatively impact transcription in the subsequent windows.
Therefore, we hypothesized that (1) Aligning specific tokens to its time information and (2) localize those hindering factors and eliminate(or substitute) them will likely to boost the accuracy on long-form setting.


## Method
In order to increment prediction accuracy on timestamp tokens for long-from audio, it is crucial to refine given audio. We define filler words and silence(Deactivated part of the audio) as a needless features and decided to eliminate them from given input. Here, a token-specific timestamp is essential for determining whether a specified token is needless or not.

<p align="center">
  <img src="/Figure/pipeline.png" width="50%" alt="Figure 1">
</p>

ASR model powered by Amazon predicts token-level timestamp from original audio. Processing pipeline, consists of filler words elimination and silence substitution, then utilizes time information to remove needless features. OpenAI’s Whisper uses refined audio data as an input, produces tokens with highly accurate timestamp which can enable model to deal with long-form audio.

### Forced Alignment
Therefore we utilized ASR model powered by Amazon, which has empirically-proven strength in forced alignment. Considering this ability, it can successfully provide time-related information which can help eliminate needless features. Audio with time-related information added now subsequently passes through processing pipeline which consists of filler words elimination and silence substitution.

### Filler words Elimination
Filler words like “um” or “uh” are common in spontaneous speech. It is desirable to detect and remove them in recordings, as they affect speech recognition accuracy (If the model sets the starting point of the context window to a strange location due to filler words interference, this affects all subsequent windows, resulting in poor overall accuracy.) and it has the potential to significantly speed up speech recognition workflows. We were able to locate filler words and remove them accurately in ms(millisecond) units by leveraging time-related information given by prior process(AWS model).

### Silence Substitution
Considering movies and lectures, long-form audio inevitably contain a lot of silence. Since silences are not considered as proper speech, it is important for speech recognition process to accurately localize and eliminate them.
There were several things to consider at this stage, as follows. (1) How to define ‘silence’ from audio, (2) Whether to simply eliminate or replace detected silences and (3) if we replace it, what will we replace it with? To find the optimal setting, we tried a lot of combinations of answers to those three questions. After all, it was experimentally confirmed that if the audio lasts more than 100ms at -45dB or less, replacing it with 250ms silence showed the best performance. Especially replacing silence with a proper interval rather than simply eliminating it showed remarkable results, which suggests that it prevented overlapping between tokens caused by suddenly disappeared silence.
After all these process, original audio contains time-related information and reduces useless information, making it more informative and efficient. It soon passes through Whisper to produce output and Fig 1 describes our whole process.

## Evaluation
In order to measure more accurate results, we collected a long-form audio data that was close to reality. We collected various data on the conversation between the two people in about 30 minutes, and produced ground truth by listening to the conversation and writing it down manually. Our metric for the experiment, WER(Word Error Rate), was calculated using a self-made ground truth and a token generated by the model according to the equation below.

<p align="center">
  <img src="/Figure/table2.png" width="50%" alt="Table 2">
</p>

Table 2 shows our experiment result, which describes result of our approach and ablation study at the same time. We can see that our approach results in a much lower WER rate than using Whisper alone, and that applying either eliminating filler words or replacing silence significantly improves WER results for long-form audio. This suggests that hindering factors such as filler words and silence was a main cause of the decline in translation performance in the process of Whisper moving the context window based on timestamp to transcribe long-form audio, which means our hypothesis was correct. Especially, when using Whisper alone, it frequently falls into repetitive loops compared to other 3 conditions, which suggests that accurate transcription has been made in subsequent windows by and accurately predicting timestamp tokens.

<p align="center">
  <img src="/Figure/figure2.png" width="50%" alt="Figure 2">
</p>

## Conclusion
The contribution of our work can be categorized into two. (1) In order to utilize the ability of the AWS model to stably align the timestamp and the accuracy of Whisper at the same time, exact timestamp for each token was extracted and used for preprocessing. (2) In the pre-processing process, long-form audio transcription accuracy was largely improved by removing filler words and silences that interfere with the process of moving context windows to process long-form audio. Given all these facts, we can conclude that audio processing using time-related information has a significant impact on the performance improvement of the ASR model.
After all, since it has been shown in Table 2 that strategy most effectively lowers WER varies for each audio, we leave designing a model equipped with preprocessing techniques based on unique characteristics for each audio as a future work.
