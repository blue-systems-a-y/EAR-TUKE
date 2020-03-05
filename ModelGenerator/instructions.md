# Generating Model for Eartuke

## Download and compile HTK

1. Download the HTK from [here](http://htk.eng.cam.ac.uk/download.shtml).

2. Run the following commands:

```bash
cd htk
cd HLMLib
make -f MakefileCPU all
cd ../HTKLib
make -f MakefileCPU all
cd ../HLMTools
make -f MakefileCPU all
cd ../HTKTools
make -f MakefileCPU all
make -f MakefileCPU install
```

## Labeling .wav files

We should "mark" the starting and ending of the acoustic event in every .wav file.
We do this with the HSLab tool provided by HTK.

1. Get a .wav file with a gunshot sound (named as gunshot_1.wav for exmaple).

2. Run the following command:

    ```bash
    cd htk
    cd HTKTools
    ./HSLab -F WAV <path_to_file>/gunshot_1.wav
    ```

    It will open a GUI with the waveform graph.

3. You should click the mark button and mark the beginning and ending of the gun shot sound, then, click on the labels button and type "gunshot". Label the background noise (before and after the gunshot) as "background".

4. Click save.

You will get a .lab, this is the label of the .wav file. If you done it correctly the file should contain something similar to:

```text
22222 4904989 background
4882540 14180045 gunshot
14180045 15272562 background
```

## Feature extraction

Although you can use HTK for feature extraction, eartuke uses its own algorithm for this task. The program that we will use is ```frontend```.

In order to run ```frontend``` we should provide a configuration file:

```bash
# frontend.cfg
ZERO_COEF T
HAMMING 0.46
ENERGY F
RAW_ENERGY F
WND_LENGTH 25
WND_SHIFT 10
PREEM 0.97
LIFT_COEF 22
ACC_WND 2
DEL_WND 2
# CEP_NUM 12
# HI_FREQ 4000
# LO_FREQ 100
MEL_NUM 12
# CMN_WND 0
FRONT_END_TYPE MELSPEC

```

- *ZERO_COEF*: If true, computes also the zero mfcc coefficient (default value = F). The zeroth coefficient is often excluded since it represents the average log-energy of the #input signal, which only carries little speaker-specific information.
- *HAMMING*: Hamming window coefficient on the input frames (default = 0.46). The hamming window is a function used to smooth the edges of the data window.
- *ENERGY*: If true, uses the avarage energy of the window as a feature (default value = F).
- *RAW_ENERGY*: If true, uses the avarage energy of the window be fore the hamming window and preemphasis as a feature. Usualy best to not include that feature (default value = F).
- *WND_LENGTH*: The window length (default value = 25 milisec).
- *WND_SHIFT*: The window shift (default value = 10 milisec).
- *PREEM*: Preemphasis coefficient. Used to balance the frequency spectrum since high frequencies usually have smaller magnitudes compared to lower frequencies (default = 0.97).
- *LIFT_COEF*: Used to "normalize" the MFCC coefficients (default = 0.97).
- *ACC_WND*: Number of frames in the energy acceleration coefficients computations (default = 2).
- *DEL_WND*: Number of frames in the energy delta coefficients computations (default = 2).
- *CEP_NUM*: Number of cepstral coefficients after cosine transform (default = 12).
- *HI_FREQ* and *LO_FREQ*: Band pass filter in Hz (default: 0 and inf, meaning no filter).
- *MEL_NUM*: Number of Mel-Bank filters (default = 29). Some articles recommend this number to be 40.
- *CMN_WND*: Cepstral mean normalization window length. This is a local normalization window. Used in some speech recognition applications as 300 frames (default = 0) **Changing this value will crush the frontend program, maybe a bug**.
- *FRONT_END_TYPE*: Type of features to compute, can be one of: MELSPEC, FBANK, MFCC, DIRECT.

To extract the features run the following command:

```bash
./frontend features.cfg <path>/gunshot_1.wav <path>/gunshot_1.features
```

You will get the output features in the file "gunshot_1.features".

## HMM Definition

Now that we prepared all the necessary data, we should define the topology of the Hidden Markov Model for every acoustic event. Hidden Markov Model is a state machine with probabilities over the edges. In edition every state outputs an observation when entered.

In this instruction we are creating a classifier for two acoustic events: `background` and `gunshot`. So we will define two HMMs.

We should choose:

- Number of states
- Form of the observation function for each state.
- Transitions between states.

There is no basic rule for defining the HMM. But we will go for 6 states chain as shown:

gunshot.hmm:

```text
~o <VecSize> 39 <MELSPEC_D_A_0>
~h "gunshot"
<BeginHMM>
<NumStates> 6
<State> 2
<Mean> 39
0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0
0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0
0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0
<Variance> 39
1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0
1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0
1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0
<State> 3
<Mean> 39
0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0
0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0
0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0
<Variance> 39
1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0
1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0
1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0
<State> 4
<Mean> 39
0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0
0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0
0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0
<Variance> 39
1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0
1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0
1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0
<State> 5
<Mean> 39
0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0
0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0
0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0 0.0
<Variance> 39
1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0
1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0
1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0 1.0
<TransP> 6
0.0 0.5 0.5 0.0 0.0 0.0
0.0 0.4 0.3 0.3 0.0 0.0
0.0 0.0 0.4 0.3 0.3 0.0
0.0 0.0 0.0 0.4 0.3 0.3
0.0 0.0 0.0 0.0 0.5 0.5
0.0 0.0 0.0 0.0 0.0 0.0
<EndHMM>
```

background.hmm should have the same form.

The files `gunshot.hmm` and `background.hmm` should be stored in a folder named `proto`.

## HMM Training

Prerequisite:

- Create an empty folder named `hmm0` for the initial HMM model.
- Create a folder named `proto` with the two .hmm files: `gunshot.hmm` and `background.hmm`.
- Create a text file named trainlist.txt that contains your paths to the `.features` files. For example:

```text
<path to data folder>/gunshots/gunshot_1.features
<path to data folder>/gunshots/gunshot_2.features
<path to data folder>/gunshots/gunshot_3.features

```

- Put your `.lab` files in a folder named `lab`

Now We will use the HInit program to create an initialized HMM. Run the following command:

```bash

./HInit -A -D -T 1 -S trainlist.txt -M hmm0 -H proto/gunshot.hmm -l gunshot -L ../lab gunshot

```

Where:

- `-l gunshot` indicates which labelled segment must be used within the training.
- And the last parameter is the name of HMM that we want to initialize.

Do this again for `background.hmm`.

Now we have two initialized HMM in the `hmm0` folder.

### Training iterations using HRest

We use the program `HRest` to perform the actuall training iterations. We need to run the following command several times:

```bash

HRest -A -D -T 1 -S trainlist.txt -M hmmi -H hmmi-1/hmmfile -l label -L label_dir nameofhmm

```
Where:

- *nameofhmm* is the name of the HMM to train (here: yes, no, or sil).
- *hmmfile* is the description file of the HMM called nameofhmm. It is stored in a directory whose name indicates the index of the last iteration (here model/hmmi-1/ for example).
- *trainlist.txt* gives the complete list of the .featues files forming the training data.
- *label_dir* is the directory where the label files (.lab) corresponding to the training data.
- *label* indicates the label to use within the training data (background or gunshot)
- *hmmi* is the output directory, indicates the index of the current iteration i.

For example:

```bash

HRest -A -D -T 1 -S trainlist.txt -M hmm1 -H hmm0/gunshot.hmm -l gunshot -L ../gunshots/ gunshot

HRest -A -D -T 1 -S trainlist.txt -M hmm2 -H hmm1/gunshot.hmm -l gunshot -L ../gunshots/ gunshot

HRest -A -D -T 1 -S trainlist.txt -M hmm3 -H hmm2/gunshot.hmm -l gunshot -L ../gunshots/ gunshot

```

TODO: write a python script that run this.

At the end of the iterative process we have two trained HMMs. Now we can concatenate their representation into a single `model.mmf` file.
