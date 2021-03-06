# A gentle introduction to LSTM,GRU and Encoder-Decoder with Attention



Recurrent Neural Networks are very good at processing sequential data like text and time series data . In a typical RNN cell , input at time step t-1 xₜ₋₁ is fed into the network and its output yₜ₋₁ is fed back to the network along with the input for time step t, xₜ . The RNN cell acts as a memory cell since its output at any given time step is a function of information from previous time steps . An RNN unrolled over time looks like below

![Unrolled RNN Cell ](https://raw.githubusercontent.com/ravindrabharathi/DL-Projects/master/images/RNN.png)

##### Problems with RNN - Short term memory :

RNN works well when the sequences are not very long. For long sequences it tends to forget initial information. A little bit of the earlier information is lost as transformation of data happens at every time step as we traverse through the network. LSTM and GRU two of the popular mechanisms by which this memory issue of RNN is addressed.

##### LSTM 

Long Term Short Term memory is the most popular mechanism for tackling RNN memory issue. LSTM is derived from the RNN cell but it adds the ability to learn what to store as long term context and what to forget. 

Unlike simple RNN , LSTM has a long term state or context c and a short term state (h) . Both c and h will be processed at each time step. LSTM gets the ability to learn what to retain for long term and what to forget using 3 gates 

a. Forget Gate : Controls what memories are not needed and so can be dropped . 

b. Input Gate : Controls which parts of the memories to be added to the long term context

c. Output gate : Controls what part of the long term state should form the short term state (and also output) for the current time step 

All three gates are fully connected layers with a sigmoid activation function with output values ranging from 0 to 1 to control what information is forgotten , added , etc. 1 would let information go through and 0 would close the gate and not let information to move forward.

A typical LSTM looks like below 

![LSTM cell](https://raw.githubusercontent.com/ravindrabharathi/DL-Projects/master/images/LSTM1.png) 





Current time step input xₜ and previous short term state hₜ₋₁ are fed to a main cell which is a fully connected layer with tanh activation function . This main cell decides what are the important parts to pass through. In a simple RNN this output from the main cell will result in the output for the current time step. But in LSTM the three other gates come into play.  

xₜ₋₁ and h ₜ₋₁ also pass through the forget gate which decides what information to forget or drop. The output from forget gate and cₜ₋₁ go through an elementwise multiplication operation as the long term context vector traverses from left to right. Here an output close to 0 from the forget cell will drop information whereas output close to 1 will keep information.

x ₜ and h ₜ₋₁ are fed to the input gate and the output from here goes through an elementwise multiplication operation with the output from main cell to form an output that contains what new information is to be added. This is added to the output from elementwise multiplication of c ₜ₋₁ and forget gate . This now forms the new long term context cₜ

A copy of the long term state cₜ is sent to through a tanh function and gets filtered by the output gate to form the short term state hₜ. This also forms the output yₜ for the current time step.



##### Gated Recurrent Unit 

GRU is similar to LSTM but has only two additional gates instead of three additional gates used in LSTM. GRU merges both cₜ and hₜ into one singe state vector hₜ . These differences mean there are fewer parameters leading to easier and faster training cycles .GRU is as effective as LSTM and so has gained in popularity. 

In addition to the usual latent state update mechanism of a fully connected layer with tanh activation function (main layer,g), there are two other gates - reset gate (r) and update gate (z).The reset gate helps in controlling how much of the previous state will be remembered. The update gate helps in controlling how much information in the hidden state should be updated with new information. 

![GRU cell](https://raw.githubusercontent.com/ravindrabharathi/DL-Projects/master/images/GRU.png) 

h ₜ₋₁ and xₜ are fed to the reset gate r which is a fully connected layer with sigmoid activation function. The output from this is elementwise multiplied with hₜ₋₁ which forms the part of information from hₜ₋₁ information that will be fed to the main layer . When output from reset gate is close to zero, all of the previous hidden state is effectively reset . And when it is close to 1 , all of the previous hidden state is retained and we get a vector (hₜ₋₁ concatenated with xₜ) similar to that of a vanilla RNN . This intermediate hidden vector is now concatenated with the current time step input xₜ and fed to the main gate to perform its usual latent update mechanism. In this way, the reset gate allows dropping of any irrelevant information and helps capture the short term dependencies. The units that capture short term dependencies will have reset gates that are frequently active.

The update gate performs the combined task of a forget gate and an input gate of LSTM . It is a fully connected layer with sigmoid activation function. Both hₜ₋₁ and xₜ are fed to the update gate . The output from this goes through a (1-) meaning if the update gate output was 1 , after the (1-) it will be (1–1=0) .This is effectively equivalent to a forget gate (of LSTM) being open and input gate (of LSTM) getting closed. If output from update gate was 0 , the effective result would be 1 and will be equivalent to input gate being open and forget gate being closed. This result from (1-) operation is elementwise multiplied by the output from the main gate and then added to the single context vector to form the new state /context vector for the current time step .In this way update gate helps capture long-term dependencies. The units that capture long term dependencies will have update gates that are very active.



##### Encoder Decoder 

Encoder -Decoder for RNN was presented in the same paper that introduced GRU cells -[Learning Phrase Representations using RNN Encoder–Decoder for Statistical Machine Translation](https://arxiv.org/abs/1406.1078) . The encoder learns to encode a variable-length sequence into a fixed-length vector representation and the decoder learns to decode a given fixed-length vector representation back into a variable-length sequence. 

An encoder - decoder looks like below . Each cell block can be an RNN / LSTM /GRU unit. LSTM or GRU is used for better performance.

![encoder-decoder](https://raw.githubusercontent.com/ravindrabharathi/DL-Projects/master/images/encoder-decoder.png)

The encoder is a stack of RNNs that encode input from each time step to context c₁,c₂, c₃ . After the encoder has looked at the entire sequence of inputs , it produces an encoded fixed length context vector c. This context vector or final hidden vector from encoder is fed to the decoder which is again a stack of RNNs. Each unit of the decoder produces an output for its time step as well as the hidden state (s) to be fed to the next time step . The decoder thus takes the fixed length vector representation from the encoder and produces the variable length output as required by the translation task .



##### Encoder -Decoder with attention 

The problem with Encoder - Decoder architecture is that the encoder has to encode the whole input sequence to be fed to the decoder and the decoder needs to figure out the individual outputs of each time step based on the entire encoded context which means that the decoder needs to find correlations considering the whole sequence and cannot just solely focus on the current time step. This becomes a problem when handling long sequences.

To overcome this issue , attention mechanism was introduced in the paper [Neural Machine Translation by Jointly Learning to Align and Translate](https://arxiv.org/abs/1409.0473) . 

![attention model](https://raw.githubusercontent.com/ravindrabharathi/DL-Projects/master/images/attention-model.png)

Here , in addition to sending the final context vector , the context or output from the encoder at each stage is also sent to the decoder. This architecture introduces an additional component called the alignment model which is a fully connected layer with a softmax output. At each time step , the outputs from the encoder pass through this alignment model that gives weightage α to the encoder output for each time step . If α for the 2nd encoder output at the 3rd decoder time step is large , the decoder will pay more attention to this. In this way the attention mechanism is able to align the sequence based on relevance at each time step and help the decoder pay attention to the more relevant parts of the sequence when making an output for a particular time step. 



