# Target Language Extraction and the Multilingual Cocktail Party
This repository comprises two main sections. Section one introduces the novel target language extraction (TLE) task and provides the data lists as well as scripts to reproduce our experiments. Section two describes the multilingual cocktail party situation and provides the information as well as scripts to simulate the new GlobalPhoneMCP-GE corpus. You can find more detailed information in our paper (please see the bottom part of this page). This repository is under an Apache 2.0 license but comprises different parts and sources with different licenses. Please check the license section for more details.

---
**Note:**  
**We will add the information and scripts to simulate the GlobalPhoneMCP-GE database and to train/test the TLE models soon.**  
**Please check this repository again in a few days. Thank you four your understanding!**  

---


## 1. Target Language Extraction
Target language extraction (TLE) attempts to extract all voices that are given in a particular target language from the remaining sounds in a given multilingual cocktail party scene. Such a TLE model is tied to isolate a specific target language regardless of the interfering language and the number of speakers in the mixture. Hence, it extracts voices based on the language rather than on the speaker characteristics. In our work, we only consider speech based signals. Each mixture comprises two languages with two or four speakers in total.


### Training Scheme
A TLE model is able to learn and embed the individual characteristics of the target language into its entire architecture and parameters. Based on those, the model estimates a mask to filter out the voices given in the target language. To accomplish this, we train the model in a supervised training scheme and provide appropriate training tuples, consisting of input and target (ground truth) data. While the input data are given as mixture signals, the ground truth data comprise either a single speaker signal or two mixed speakers. This highlights the differences compared to previous studies in which mostly a single speaker was estimated per output channel. In 2T-2I scenes, we estimate mixture speech signals as a single output stream. The tuples for the different scenes are constructed as follows:

| Scene | Input | Target (ground truth) |
| :-----: | :--- |:--- |
| 1T-1I | Mixture signal comprising two speakers (one target language speaker and one interfering language speaker) | Single speaker signal (just the single target language speaker)|
| 2T-2I | Mixture signal comprising four speakers (two target language speakers and two interfering language speakers) | Two speaker mixture signal (a mixture consisting of just the two target language speakers)|

In our experiments, we remove three interfering languages from the training set in order to test the TLE model under closed and open set conditions for interfering languages. Additionally, we test the model under open set conditions for speakers, as the test set speakers are disjoint from the training and cross-validation set speakers by design.

The implementation of this overall TLE training and testing framework is based on [4].


### Model Adaptation
We apply three recent model architectures that have been proposed for speaker extraction or blind source separation tasks. More in detail, we use the SpEx+ [1], Conv-TasNet [2], and SepFormer [3] architectures. To incorporate those into our training scheme, we have to slightly adapt their respective structures, as explained in the following.

- **SpEx+**: The SpEx+ architecture has been introduced for speaker extraction tasks. We remove the speaker encoder part since we do not use any reference signal for the TLE task. The rest of the model remains the same as given in the original implementation in [1]. We refer this modified version as SpEx+<sub>Ref.-Free</sub>. The implementation of the model is done with [4].

- **Conv-TasNet**: The Conv-TasNet architecture has been introduced for blind source separation tasks. We change the structure to estimate a single mask and change the mask activation to ReLU. The remaining parts are set according to the settings of the best model in [2]. We refer this modified version as Conv-TasNet<sub>Single-Mask</sub>. The implementation of the model is done with the Asteroid toolkit [5-6].

- **SepFormer**: The SepFormer architecture has been introduced for blind source separation tasks. We change the structure to estimate a single mask. The rest of the model is set according to the settings of the best model in [3]. We refer this modified version as SepFormer<sub>Single-Mask</sub>. The implementation of the model is done with the SpeechBrain toolkit [7-8].


## 2. Multilingual Cocktail Party
A multilingual cocktail party situation comprises overlapping multi-talker speech in different languages. Practical examples are crowded places with multilingual soundscapes, such as conferences or airports. Those scenes are different from the ones that have been usually studied in the past.


### GlobalPhoneMCP Design Concept
The creation of the multilingual cocktail party corpus utilizes the GlobalPhone 2000 Speaker Package [9]. This database comprises 2,000 speakers and covers 22 different languages in total. Each speaker is presented with roughly 40 seconds of audio data. In the following, we describe how to simulate the multilingual cocktail party situation based on the GlobalPhone 2000 Speaker Package and refer this as a general corpus design concept, called GlobalPhone Multilingual Cocktail Party (GlobalPhoneMCP). If this design concept is applied to simulate the corpus for a specific target language, an identifier is added to the corpus reference, as e.g., GlobalPhone Multilingual Cocktail Party - German (GlobalPhoneMCP-GE).

As first attempt at multilingual cocktail party situations, we define the scenarios to always comprise two languages, i.e., one target language and one interfering language. Since the GlobalPhone 2000 Speaker Package covers 22 different languages in total, we can select one language as fixed target language while the interfering language varies between the remaining 21 languages. This leads to 21 language datasets (German-Arabic, German-Portuguese, etc.). We simulate each language dataset under two conditions and explain the differences in the following.

- **1T-1I scene**: This condition simulates multilingual cocktail party situations with two speakers per mixture. One speaker is talking in the target language while the other one is talking in the interfering language. The possible signal-to-noise ratio (SNR) between two speakers ranges in an interval of [-5 to 5] dB. Whether the target language speaker or the interfering language speaker is in the fore- or background is randomly chosen to reflect real world situations.

- **2T-2I scene**: This condition simulates multilingual cocktail party situations with four speakers per mixture. Two speakers are talking in the target language while two other speakers are talking in the interfering language. To create those scenes, we separately mix two target language speakers and two interfering speakers and apply the same procedure as for the 1T-1I scene. We consider those as target language mixture and interfering language mixture. Finally, we mix those two mixtures by following the 1T-1I procedure again. This leads to the final mixture which comprises four speakers. We always selected the fore- and background speaker/mixture randomly. The final possible signal-to-noise ratio (SNR) between two speakers ranges in an interval of [-10 to 10] dB.  

The general speaker and utterance split into training (tr), cross-validation (cv), and test (tt) data is the same as for our previously proposed GlobalPhoneMS2 corpus (https://github.com/mborsdorf/GlobalPhoneMS_Scripts). While the speakers in the training and cross-validation sets are the same but with different utterances, the speakers in the test set are disjoint from those to simulate an open set test condition.


### About GlobalPhoneMCP-GE
In our work, we consider German as fixed target language while the interfering language varies between the remaining 21 languages under 1T-1I and 2T-2I conditions. We apply the aforementioned design concept and create the GlobalPhone Multilingual Cocktail Party - German (GlobalPhoneMCP-GE) corpus. To the best of our knowledge, this represents the first of its kind multilingual cocktail party database for speech separation. We utilize each German utterance four times when mixing German with an interfering language to have a meaningful corpus size.


### Data Simulation
To simulate the actual data, we adapt the scripts which have been released to create the wsj0-2mix database as used in [10]. For the original scripts please consult [11]. We apply the scripts to always mix two speakers at a time. That means, we can directly mix the data for the 1T-1I condition while the 2T-2I condition requires three separate steps (as described under "GlobalPhoneMCP Design Concept"). We simulate the data with 8 kHz sampling frequency and as min version. The latter means that the longer utterances/mixture is truncated to the length of the shorter one when two utterances/mixtures are mixed.


## Our Paper
If you enjoyed working with the Target Language Extraction approach, the GlobalPhoneMCP design concept, or the GlobalPhoneMCP-GE corpus, please cite us:
```
@INPROCEEDINGS{Borsdorf2021ASRU,
  author={Borsdorf, Marvin and Li, Haizhou and Schultz, Tanja},
  booktitle={2021 IEEE Automatic Speech Recognition and Understanding Workshop (ASRU)},
  title={Target Language Extraction at Multilingual Cocktail Parties},
  year={2021},
  volume={},
  number={},
  pages={717-724},
  doi={10.1109/ASRU51503.2021.9688052}}
```



## References
[1]  M. Ge, C. Xu, L. Wang, E. S. Chng, J. Dang, and H. Li, “SpEx+: A Complete Time Domain Speaker Extraction Network,” in Proceedings of the Annual Conference of the International Speech Communication Association (INTERSPEECH), 2020, pp. 1406–1410.  
[2] Y. Luo and N. Mesgarani, “Conv-TasNet: Surpassing Ideal Time–Frequency Magnitude Masking for Speech Separation,” in IEEE/ACM Transactions on Audio, Speech, and Language Processing, vol. 27, pp. 1256–1266, 2019.  
[3] C. Subakan, M. Ravanelli, S. Cornell, M. Bronzi, and J. Zhong, “Attention Is All You Need In Speech Separation,” in Proceedings of the IEEE International Conference on Acoustics, Speech, and Signal Processing (ICASSP), 2021, pp. 21–25.  
[4] https://github.com/gemengtju/SpEx_Plus  
[5] M. Pariente, S. Cornell, J. Cosentino, S. Sivasankaran, E. Tzinis, J. Heitkaemper, M. Olvera, F.-R. Stöter, M. Hu, J. M. Martín-Doñas, D. Ditter, A. Frank, A. Deleforge, and E. Vincent, “Asteroid: The PyTorch-based Audio Source Separation Toolkit for Researchers,” in Proceedings of the Annual Conference of the International Speech Communication Association (INTERSPEECH), 2020, pp. 2637–2641.  
[6] https://github.com/asteroid-team/asteroid  
[7] M. Ravanelli, T. Parcollet, P. Plantinga, A. Rouhe, S. Cornell, L. Lugosch, C. Subakan, N. Dawalatabad, A. Heba, J. Zhong, J.-C. Chou, S.-L. Yeh, S.-W. Fu, C.-F. Liao, E. Rastorgueva, F. Grondin,W. Aris, H. Na, Y. Gao, R. D. Mori, and Y. Bengio, “SpeechBrain: A General-Purpose Speech Toolkit,” arXiv preprint arXiv:2106.04624v1, 2021.  
[8] https://github.com/speechbrain/speechbrain  
[9] https://catalog.elra.info/en-us/repository/browse/ELRA-S0400/  
[10] J. R. Hershey, Z. Chen, J. Le Roux, and S. Watanabe, "Deep Clustering: Discriminative Embeddings for Segmentation and Separation", in Proceedings of the IEEE International Conference on Acoustics, Speech, and Signal Processing (ICASSP), 2016, pp. 31-35.  
[11] Y. Isik, J. Le Roux, S. Watanabe Z. Chen, and J. R. Hershey, “Scripts to Create wsj0-2 Speaker Mixtures,” MERL Research. Retrieved June 2, 2020, from https://www.merl.com/demos/deep-clustering/create-speaker-mixtures.zip, [Online].  


## Licenses
- The SpEx+ repository, that is the basis of this overall training and testing framework implementation, is under the MIT license, see [4].
- The Asteroid toolkit is under the MIT license, see [6].
- The SpeechBrain toolkit is under the Apache 2.0 license, see [8].
- The mixing scripts for the GlobalPhoneMCP design concept and to generate GlobalPhoneMCP-GE are under the Apache 2.0 license, see [11].
- You can get access to the GlobalPhone 2000 Speaker Package by purchasing a research or commercial license, see [9].
