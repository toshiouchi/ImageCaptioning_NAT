# We report a test data of image captioning with Non-Autoregressive Transformer ( NAT ).

In general, non-autoregressive transformers using cross-entropy loss are known to result 
in repeated tokens in the resulting sentences.

https://arxiv.org/abs/2207.04206

We also confirmed this in image captioning using the clip + bert + CEloss system. 
For this reason, we decided to use CTCloss for loss this time. 

## Two measurements

A clip + transformer decoder + CTCLoss system has been measured. Moreover, a system 
with bert KL Divergence loss has been measured.

### Clip + Transformer Decoder + CTCLoss system.

An image is input to CLIP. CLIP is a Feature Extractor + Encoder. 
The output of CLIP is sent to a Dense Connector so that it can be input to a 
Transformer Decoder. The target input of the Transformer Decoder is 
the output of the Dense Connector downsampled in the sequence direction. 
The memory input is the output of the Dense Connector. These are input to a 
Transformer Decoder, and the output is logits. A CTCloss is calculated with the logits of transoformer 
decoder and the teacher captions. The transformer decoder has 36 layers, heads = 20, and 
dim_hidden = 1280.

<img width="570" height="710" alt="Diagram_Decoder_Only_en" src="https://github.com/user-attachments/assets/706bfe79-bcc9-4033-8009-b2a6af0d8b77" />

## With bert system.

The process is the same as the system above, up to the point where an image is input to 
CLIP and CTC Loss is obtained. Since we want to provide BERT information to this CTC Loss system, 
we place an auxiliary Transformer Decoder and Bert as shown in the figure. 
We take the KL divergence loss of the output logits of the auxiliary Transformer Decoder and 
the output logits of Bert, and add this to CTC Loss to determine the overall loss. 
The Transformer Decoder has 24 layers, heads = 16, and dim_hidden = 1024, to match BERT large.

<img width="1336" height="783" alt="Diagram_Bert_KD_en" src="https://github.com/user-attachments/assets/3a76ad9b-54cf-49cf-96da-3999322a3a4b" />

## Results

We trained both systems for 10 epochs on the v7 dataset, and then generated 
captions using 21 test data sets.

### Added October 30, 2025.
The Decoder Only measurement results were for a Transformer Decoder with 36 layers, heads = 20, dim_hidden = 1280, and a total parameter count of approximately 1B. This time, we are adding measurement results for a total parameter count of approximately 2.2B, with 36 layers, 24 heads, and embedding_dim = 2304. 

The bert corrected hypo in the 2.2B measurement result is the normal inference result shown in the hypothesis corrected by Bert. WER2 and BLER2 are values ​​related to this bert corrected hypo.

```
Decoder Only
hypo: in this image we can see straw on the surface .
refe: in this i can see there are red colored strawberries.
this pic. WER : 0.6666666666666666
this pic. BLEU: 0.5409165243982964
test number = 1 average, WER = 0.6666666865348816, BLEU = 0.5409165024757385

Bert KD
hypo: in this image we can see straw ##berries surface .
refe: in this i can see there are red colored strawberries.
this pic. WER : 0.5833333333333334
this pic. BLEU: 0.6075729944517408
test number = 1 average, WER = 0.5833333134651184, BLEU = 0.6075729727745056

Decoder Only 2.2B
reference          : in this i can see there are red colored strawberries.
hypothesis         : in this image we can see strawberries surface.
bert corrected hypo: in this image we can see strawberries growing.
this pic. WER  : 0.5833333333333334
this pic. BLEU : 0.6297979014940561
this pic. WER2 : 0.5833333333333334
this pic. BLEU2: 0.6262188175395172
test number = 1 average, WER = 0.5833333134651184, WER2 = 0.5833333134651184, BLEU = 0.6297978758811951, BLEU2 = 0.6262187957763672
```
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2958180/72e86a2a-7f1a-41ae-8bf5-b3771e7ab7c7.png)
　
```
Decoder Only
hypo: in this image we can people sitting on chair and screens .
refe: there are few persons sitting on the chairs. here we can see monitors, keyboards, tables, and devices.
this pic. WER : 0.8181818181818182
this pic. BLEU: 0.3198485665828107
test number = 2 average, WER = 0.7424242496490479, BLEU = 0.43038254976272583

Bert KD
hypo: in this image we can see a people sitting on chair and there are people table .
refe: there are few persons sitting on the chairs. here we can see monitors, keyboards, tables, and devices.
this pic. WER : 0.9090909090909091
this pic. BLEU: 0.5389700256269433
test number = 2 average, WER = 0.7462121248245239, BLEU = 0.5732715129852295

Decoder Only 2.2B
reference          : there are few persons sitting on the chairs. here we can see monitors, keyboards, tables, and devices.
hypothesis         : in this image we can see a woman sitting on there are people monitors.
bert corrected hypo: in this image we can see a woman sitting on there are two flowers.
this pic. WER  : 0.8181818181818182
this pic. BLEU : 0.4962872741812295
this pic. WER2 : 0.8181818181818182
this pic. BLEU2: 0.41440482624892505
test number = 2 average, WER = 0.7007575631141663, WER2 = 0.7007575631141663, BLEU = 0.5630425810813904, BLEU2 = 0.5203118324279785


```
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2958180/ba295e00-927d-472c-9358-27bd90975243.png)
　
```
Decoder Only
hypo: in this image can are three people standing and . in the background there can buildings building .
refe: this image is taken outdoors. at the bottom of the there is a floor. in the background there are a few buildings with walls, windows and balconies. in the middle of the image two men and a woman are standing on the floor and they are with smiling faces.
this pic. WER : 0.8571428571428571
this pic. BLEU: 0.1759781025429421
test number = 3 average, WER = 0.780663788318634, BLEU = 0.3455810546875

Bert KD
hypo: in this image we can see three people standing bag . in the background there can buildings building .
refe: this image is taken outdoors. at the bottom of the there is a floor. in the background there are a few buildings with walls, windows and balconies. in the middle of the image two men and a woman are standing on the floor and they are with smiling faces.
this pic. WER : 0.8571428571428571
this pic. BLEU: 0.17823349739392516
test number = 3 average, WER = 0.783189058303833, BLEU = 0.44159218668937683

Decoder Only 2.2B
reference          : this image is taken outdoors. at the bottom of the there is a floor. in the background there are a few buildings with walls, windows and balconies. in the middle of the image two men and a woman are standing on the floor and they are with smiling faces.
hypothesis         : in this image we can see three people standing. in the background there can buildings buildings.
bert corrected hypo: in this image we can see three people standing. in the background there can buildings standing.
this pic. WER  : 0.8571428571428571
this pic. BLEU : 0.1570963482348411
this pic. WER2 : 0.8392857142857143
this pic. BLEU2: 0.15439511922613536
test number = 3 average, WER = 0.7528859972953796, WER2 = 0.74693363904953, BLEU = 0.42772719264030457, BLEU2 = 0.3983395993709564

```
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2958180/7ab532c2-7ec8-4e50-80dc-a78f4d6c0919.png)
　
```
Decoder Only
hypo: in this image we can see plants flower . there is an insect . the there is sky .
refe: in this image in front there are plants. in the background of the image there is sky.
this pic. WER : 0.5789473684210527
this pic. BLEU: 0.6441286229283673
test number = 4 average, WER = 0.730234682559967, BLEU = 0.42021793127059937

Bert KD
hypo: in this image we can see plants there can insect . in the background there can clouds .
refe: in this image in front there are plants. in the background of the image there is sky.
this pic. WER : 0.5789473684210527
this pic. BLEU: 0.7190262604342581
test number = 4 average, WER = 0.7321286201477051, BLEU = 0.5109506845474243

Decoder Only 2.2B
reference          : in this image in front there are plants. in the background of the image there is sky.
hypothesis         : in this image we can see plants there is a insect. the background there can sky.
bert corrected hypo: in this image we can see plants there is a earth. the background there can sky.
this pic. WER  : 0.5789473684210527
this pic. BLEU : 0.7449121091419013
this pic. WER2 : 0.5789473684210527
this pic. BLEU2: 0.7402001616448848
test number = 4 average, WER = 0.7094013094902039, WER2 = 0.7049370408058167, BLEU = 0.5070233941078186, BLEU2 = 0.48380473256111145

```
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2958180/8f697685-d364-4234-8507-9f6e976e3f7f.png)
　

```
Decoder Only
hypo: in this image we can see a woman wearing black scarf .
refe: this is a black and white image. in this image we can see women wearing spectacles.
this pic. WER : 0.631578947368421
this pic. BLEU: 0.5294512659233979
test number = 5 average, WER = 0.7105035185813904, BLEU = 0.4420646131038666

Bert KD
hypo: in this white image we can see a woman wearing wearing scarf scarf . .
refe: this is a black and white image. in this image we can see women wearing spectacles.
this pic. WER : 0.7894736842105263
this pic. BLEU: 0.6331006075369835
test number = 5 average, WER = 0.7435976266860962, BLEU = 0.5353806614875793

Decoder Only 2.2B
reference          : this is a black and white image. in this image we can see women wearing spectacles.
hypothesis         : in this black image we can see a woman wearing.
bert corrected hypo: in this black image we can see a woman wearing.
this pic. WER  : 0.6842105263157895
this pic. BLEU : 0.4469383153788137
this pic. WER2 : 0.6842105263157895
this pic. BLEU2: 0.4469383153788137
test number = 5 average, WER = 0.7043631672859192, WER2 = 0.7007917165756226, BLEU = 0.49500641226768494, BLEU2 = 0.4764314591884613

```
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2958180/c28eb13b-cdc8-4c2b-bd0b-fab54cc6853a.png)
　
```
Decoder Only
hypo: in this image we can see a food in plate .
refe: as we can see in the image there is a white color plate. in plate there is a dish.
this pic. WER : 0.7619047619047619
this pic. BLEU: 0.32737356029156794
test number = 6 average, WER = 0.7190704345703125, BLEU = 0.4229494333267212

Bert KD
hypo: in this image we can see a food in the plate .
refe: as we can see in the image there is a white color plate. in plate there is a dish.
this pic. WER : 0.8095238095238095
this pic. BLEU: 0.37850851318496903
test number = 6 average, WER = 0.754585325717926, BLEU = 0.5092353224754333

Decoder Only 2.2B
reference          : as we can see in the image there is a white color plate. in plate there is a dish.
hypothesis         : in this image we can see a food on a plate.
bert corrected hypo: in this image we can see a food on a plate.
this pic. WER  : 0.8095238095238095
this pic. BLEU : 0.3183245123380126
this pic. WER2 : 0.8095238095238095
this pic. BLEU2: 0.3183245123380126
test number = 6 average, WER = 0.7218899726867676, WER2 = 0.7189137935638428, BLEU = 0.4655594527721405, BLEU2 = 0.4500803053379059

```
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2958180/29778bf9-1412-437c-8aef-a16887cfacc6.png)
　
```
Decoder Only
hypo: in this image a woman guitar holding a guitar there is a wall .
refe: this image is clicked in a musical concert where there is a woman standing and she is holding a guitar in her hand. she is wearing black color dress. there is a mic in front of her and there is a bottle. she is holding a stick. there are speakers back side and there are some musical instruments on the bottom left corner.
this pic. WER : 0.8529411764705882
this pic. BLEU: 0.014975101608451207
test number = 7 average, WER = 0.7381948232650757, BLEU = 0.3646673858165741

Bert KD
hypo: in this image a woman holding a guitar guitar there is wall .
refe: this image is clicked in a musical concert where there is a woman standing and she is holding a guitar in her hand. she is wearing black color dress. there is a mic in front of her and there is a bottle. she is holding a stick. there are speakers back side and there are some musical instruments on the bottom left corner.
this pic. WER : 0.8676470588235294
this pic. BLEU: 0.01268065618321168
test number = 7 average, WER = 0.7707370519638062, BLEU = 0.43829894065856934

Decoder Only 2.2B
reference          : this image is clicked in a musical concert where there is a woman standing and she is holding a guitar in her hand. she is wearing black color dress. there is a mic in front of her and there is a bottle. she is holding a stick. there are speakers back side and there are some musical instruments on the bottom left corner.
hypothesis         : in this image we can a woman holding a guitar. there is musical is is wall.
bert corrected hypo: in this image we can a woman holding a guitar. there is there is is there.
this pic. WER  : 0.8235294117647058
this pic. BLEU : 0.034837457095115376
this pic. WER2 : 0.7941176470588235
this pic. BLEU2: 0.033696343603897505
test number = 7 average, WER = 0.7364098429679871, WER2 = 0.7296571731567383, BLEU = 0.4040277302265167, BLEU2 = 0.390596866607666

```
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2958180/27408da8-df9d-48e7-9919-6cd9de322462.png)
　
```
Decoder Only
hypo: in this image we can see a dog . in the background there is blur snow .
refe: in this image, we can see a black color dog, there is a blurred background.
this pic. WER : 0.5
this pic. BLEU: 0.7534088300060641
test number = 8 average, WER = 0.70842045545578, BLEU = 0.4132600426673889

Bert KD
hypo: in this image we can see a dog . in the background there is snow .
refe: in this image, we can see a black color dog, there is a blurred background.
this pic. WER : 0.5
this pic. BLEU: 0.6988017986200699
test number = 8 average, WER = 0.7368949055671692, BLEU = 0.4708617925643921

Decoder Only 2.2B
reference          : in this image, we can see a black color dog, there is a blurred background.
hypothesis         : in this image we can see a dog. the background can blurred.
bert corrected hypo: in this image we can see a dog. the background is black.
this pic. WER  : 0.4444444444444444
this pic. BLEU : 0.681201335578254
this pic. WER2 : 0.5
this pic. BLEU2: 0.6533050183622708
test number = 8 average, WER = 0.6999142169952393, WER2 = 0.7009499669075012, BLEU = 0.4386743903160095, BLEU2 = 0.42343541979789734
```
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2958180/9e5c5e7a-edd3-43c7-82ec-eef53715efcf.png)
　
```
Decoder Only
hypo: in this image we can see a food on food in bowl . spoon on on there can table .
refe: in this picture there is a bowl and a plate in the center of the image, which contains food items in it.
this pic. WER : 0.8333333333333334
this pic. BLEU: 0.4239862704579267
test number = 9 average, WER = 0.7222996354103088, BLEU = 0.41445186734199524

Bert KD
hypo: in this image we can see a food on the bowl , and . the there can see a the table .
refe: in this picture there is a bowl and a plate in the center of the image, which contains food items in it.
this pic. WER : 0.7916666666666666
this pic. BLEU: 0.49990321745822686
test number = 9 average, WER = 0.7429806590080261, BLEU = 0.4740886092185974

Decoder Only 2.2B
reference          : in this picture there is a bowl and a plate in the center of the image, which contains food items in it.
hypothesis         : in this image we can see a food food bowl, spoon table.
bert corrected hypo: in this image we can see a food - table, spoon table.
this pic. WER  : 0.7916666666666666
this pic. BLEU : 0.24294166468757164
this pic. WER2 : 0.7916666666666666
this pic. BLEU2: 0.2175677656210522
test number = 9 average, WER = 0.7101088762283325, WER2 = 0.7110296487808228, BLEU = 0.4169263243675232, BLEU2 = 0.40056121349334717
```
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2958180/f03fce68-d5c3-4d8e-a92d-5de5dad61d7f.png)
　
```
Decoder Only
hypo: in this image we can see a food on blur .
refe: in this picture i can observe some food places in the plate. the food is in brown, orange, green and red colors. it is looking like a burger. the background is completely blurred.
this pic. WER : 0.8717948717948718
this pic. BLEU: 0.0243064396862738
test number = 10 average, WER = 0.7372491955757141, BLEU = 0.37543731927871704

Bert KD
hypo: in this image we can see a food on the is blur .
refe: in this picture i can observe some food places in the plate. the food is in brown, orange, green and red colors. it is looking like a burger. the background is completely blurred.
this pic. WER : 0.8205128205128205
this pic. BLEU: 0.04798441076241786
test number = 10 average, WER = 0.7507338523864746, BLEU = 0.43147820234298706

Decoder Only 2.2B
reference          : in this picture i can observe some food places in the plate. the food is in brown, orange, green and red colors. it is looking like a burger. the background is completely blurred.
hypothesis         : in this image we can see a food on plate.
bert corrected hypo: in this image we can see a food on top.
this pic. WER  : 0.8461538461538461
this pic. BLEU : 0.025711460611413713
this pic. WER2 : 0.8717948717948718
this pic. BLEU2: 0.01845652499150742
test number = 10 average, WER = 0.7237133979797363, WER2 = 0.7271061539649963, BLEU = 0.37780484557151794, BLEU2 = 0.3623507618904114
```
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2958180/e99d4a70-9e0f-418c-9cfd-ab948431ce64.png)
　
```
Decoder Only
hypo: in this image we can people a on there a table the table .
refe: in this image i can see a person standing wearing a black shirt, blue jeans and glasses. he is holding a electronic gadget in his hand. in the background i can see few people standing, and the ceiling of the building.
this pic. WER : 0.8297872340425532
this pic. BLEU: 0.049831660115363975
test number = 11 average, WER = 0.745661735534668, BLEU = 0.3458368182182312

Bert KD
hypo: in this image we can see a people standing a table on table there is table . the there table .
refe: in this image i can see a person standing wearing a black shirt, blue jeans and glasses. he is holding a electronic gadget in his hand. in the background i can see few people standing, and the ceiling of the building.
this pic. WER : 0.7446808510638298
this pic. BLEU: 0.20696658578936322
test number = 11 average, WER = 0.750183641910553, BLEU = 0.4110680818557739

Decoder Only 2.2B
reference          : in this image i can see a person standing wearing a black shirt, blue jeans and glasses. he is holding a electronic gadget in his hand. in the background i can see few people standing, and the ceiling of the building.
hypothesis         : in this image we can see people. there is a table. the background there is wall.
bert corrected hypo: in this image we can see everything. there is a table. the background there is black.
this pic. WER  : 0.7446808510638298
this pic. BLEU : 0.14917195367333866
this pic. WER2 : 0.7446808510638298
this pic. BLEU2: 0.1695435923884506
test number = 11 average, WER = 0.7256194949150085, WER2 = 0.7287037372589111, BLEU = 0.3570200204849243, BLEU2 = 0.34482285380363464

```
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2958180/b7a008a5-d5c3-4cc0-81e0-5889be4bb80e.png)
　
```
Decoder Only
hypo: in this image we can see car there are banners banners banners . background there can buildings the background and sky .
refe: in this image we can see cars, people, banners, hoardings, tent, pole, trees, boards, and buildings. in the background there is sky.
this pic. WER : 0.6470588235294118
this pic. BLEU: 0.6637801792014791
test number = 12 average, WER = 0.7374448776245117, BLEU = 0.3723320960998535

Bert KD
hypo: in this image we can see cars the are banners there can banners . in the background there are buildings and the sky .
refe: in this image we can see cars, people, banners, hoardings, tent, pole, trees, boards, and buildings. in the background there is sky.
this pic. WER : 0.6470588235294118
this pic. BLEU: 0.692156846952336
test number = 12 average, WER = 0.7415898442268372, BLEU = 0.4344921410083771

Decoder Only 2.2B
reference          : in this image we can see cars, people, banners, hoardings, tent, pole, trees, boards, and buildings. in the background there is sky.
hypothesis         : in this image we can see car camera. there people camera. there is. the there are and sky.
bert corrected hypo: in this image we can see car park. there is sky. there is. the there are and sky.
this pic. WER  : 0.7058823529411765
this pic. BLEU : 0.4534163557631265
this pic. WER2 : 0.7058823529411765
this pic. BLEU2: 0.4079393209586292
test number = 12 average, WER = 0.7239747643470764, WER2 = 0.7268021106719971, BLEU = 0.36505305767059326, BLEU2 = 0.35008254647254944
```
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2958180/0b66890d-1f53-4bd0-ade6-2cf276bb8879.png)
　
```
Decoder Only
hypo: in this image there are two men standing on is holding a gun . the background there are trees , . the there are and trees .
refe: in this image we can see two persons standing and holding the objects, there are some stones, grass, plants and trees, also we can see the sky.
this pic. WER : 0.75
this pic. BLEU: 0.575617142838839
test number = 13 average, WER = 0.7384106516838074, BLEU = 0.38796937465667725

Bert KD
hypo: in this image we can see two men standing the is holding a gun . the there are trees , trees .
refe: in this image we can see two persons standing and holding the objects, there are some stones, grass, plants and trees, also we can see the sky.
this pic. WER : 0.625
this pic. BLEU: 0.5201348862893668
test number = 13 average, WER = 0.732621431350708, BLEU = 0.4410800337791443

Decoder Only 2.2B
reference          : in this image we can see two persons standing and holding the objects, there are some stones, grass, plants and trees, also we can see the sky.
hypothesis         : in this image we can see two men standing is holding a gun there are. the background there are stones and trees.
bert corrected hypo: in this image we can see two men standing is holding a gun and them. the background there are stones and trees.
this pic. WER  : 0.5625
this pic. BLEU : 0.6360396439203923
this pic. WER2 : 0.625
this pic. BLEU2: 0.6374162044781478
test number = 13 average, WER = 0.7115536332130432, WER2 = 0.7189711928367615, BLEU = 0.3858981728553772, BLEU2 = 0.372185081243515
```
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2958180/789ec1ac-1aba-42af-a4b6-20ae28a5d90b.png)
　
```
Decoder Only
hypo: in this image we can see a heads and there are . in the there are trees .
refe: in front of the image there are some engravings on the headstone, around the headstone on the surface there are green leaves and dry leaves and sticks, behind the headstone there are trees and a wall.
this pic. WER : 0.7619047619047619
this pic. BLEU: 0.14851496865804886
test number = 14 average, WER = 0.7400888204574585, BLEU = 0.37086552381515503

Bert KD
hypo: in this image we can see a stone there plants plants . in the background there are trees .
refe: in front of the image there are some engravings on the headstone, around the headstone on the surface there are green leaves and dry leaves and sticks, behind the headstone there are trees and a wall.
this pic. WER : 0.8095238095238095
this pic. BLEU: 0.2139070757304621
test number = 14 average, WER = 0.7381144762039185, BLEU = 0.4248533546924591

Decoder Only 2.2B
reference          : in front of the image there are some engravings on the headstone, around the headstone on the surface there are green leaves and dry leaves and sticks, behind the headstone there are trees and a wall.
hypothesis         : in this image we can see a stone there leaves. in the background there are trees.
bert corrected hypo: in this image we can see a house there is. in the background there are trees.
this pic. WER  : 0.7857142857142857
this pic. BLEU : 0.1863187894945934
this pic. WER2 : 0.8095238095238095
this pic. BLEU2: 0.14913518590823696
test number = 14 average, WER = 0.7168508172035217, WER2 = 0.7254392504692078, BLEU = 0.371642529964447, BLEU2 = 0.3562529385089874
```
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2958180/a54e4a4f-690b-4595-bd42-f1ca02c00fef.png)
　
```
Decoder Only
hypo: in this image can see a wall trees . the there is sky .
refe: in this picture i can see building and few trees and a cloudy sky.
this pic. WER : 0.6
this pic. BLEU: 0.48914025574620656
test number = 15 average, WER = 0.7307495474815369, BLEU = 0.3787505030632019

Bert KD
hypo: in this image we can see a wall trees trees . the there is sky .
refe: in this picture i can see building and few trees and a cloudy sky.
this pic. WER : 0.6
this pic. BLEU: 0.5075970769304151
test number = 15 average, WER = 0.7289068102836609, BLEU = 0.43036964535713196

Decoder Only 2.2B
reference          : in this picture i can see building and few trees and a cloudy sky.
hypothesis         : in this image we can see a wall trees. the there is sky.
bert corrected hypo: in this image we can see a few trees. the there is sky.
this pic. WER  : 0.6
this pic. BLEU : 0.4893033117347264
this pic. WER2 : 0.5333333333333333
this pic. BLEU2: 0.5369904450245785
test number = 15 average, WER = 0.709060788154602, WER2 = 0.7126321792602539, BLEU = 0.3794865608215332, BLEU2 = 0.3683021664619446
```
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2958180/f0bf2d78-df94-4de0-b80e-479bf4609ed4.png)
　
```
Decoder Only
hypo: in this image we can people . there buildings the sky .
refe: in this image there are many people in front of the building. some of them are holding camera. in the background there are buildings. there is a banner over here.
this pic. WER : 0.7647058823529411
this pic. BLEU: 0.12524073602103492
test number = 16 average, WER = 0.7328717708587646, BLEU = 0.36290615797042847

Bert KD
hypo: in this image we can see people holding cameras . the there can buildings cameras . in the background there is buildings sky .
refe: in this image there are many people in front of the building. some of them are holding camera. in the background there are buildings. there is a banner over here.
this pic. WER : 0.6470588235294118
this pic. BLEU: 0.6335045942414068
test number = 16 average, WER = 0.7237913608551025, BLEU = 0.4430656135082245

Decoder Only 2.2B
reference          : in this image there are many people in front of the building. some of them are holding camera. in the background there are buildings. there is a banner over here.
hypothesis         : in this image we can see people standing are cameras. in the background there can sky.
bert corrected hypo: in this image we can see people standing are cameras. in the background there is sky.
this pic. WER  : 0.6764705882352942
this pic. BLEU : 0.3363974062106665
this pic. WER2 : 0.6470588235294118
this pic. BLEU2: 0.3438840011445091
test number = 16 average, WER = 0.7070239186286926, WER2 = 0.7085337042808533, BLEU = 0.3767935037612915, BLEU2 = 0.3667760193347931

```

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2958180/61496715-a69c-4caf-9c4d-9deee0f76f0c.png)

　
```
Decoder Only
hypo: in this image we can see a snake the . there see and grass .
refe: in this image i can see a snake on the ground. it is in black color. i can see few wooden sticks, few stones and grass.
this pic. WER : 0.5666666666666667
this pic. BLEU: 0.32647241788622017
test number = 17 average, WER = 0.7230949997901917, BLEU = 0.36076298356056213

Bert KD
hypo: in this image we can see a snake on the ground and there can stones and grass .
refe: in this image i can see a snake on the ground. it is in black color. i can see few wooden sticks, few stones and grass.
this pic. WER : 0.5
this pic. BLEU: 0.559360359339568
test number = 17 average, WER = 0.7106271386146545, BLEU = 0.44990643858909607

Decoder Only 2.2B
reference          : in this image i can see a snake on the ground. it is in black color. i can see few wooden sticks, few stones and grass.
hypothesis         : in this image we can see a snake on the ground and there grass.
bert corrected hypo: in this image we can see a snake on the ground and there is.
this pic. WER  : 0.6
this pic. BLEU : 0.3864866535228235
this pic. WER2 : 0.6
this pic. BLEU2: 0.34654777821793276
test number = 17 average, WER = 0.7007283568382263, WER2 = 0.7021493911743164, BLEU = 0.3773636817932129, BLEU2 = 0.3655861020088196
```
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2958180/51e0639b-ff3d-4814-b85a-0b241f0b1844.png)

 
　

```
Decoder Only
hypo: in this image we can see a woman holding a paper there is a podium is a laptop the there is a laptop . in the background there is curtain .
refe: in this image we can see a person standing and holding a book and to the side we can see a podium with mic and there is a laptop and some other objects on the table. we can see a person standing in the bottom right.
this pic. WER : 0.6458333333333334
this pic. BLEU: 0.450435022276757
test number = 18 average, WER = 0.7188026905059814, BLEU = 0.36574479937553406

Bert KD
hypo: in this image we can see a woman holding paper there is a podium . podium there is a laptop and podium . in the background there is a laptop the there curtain .
refe: in this image we can see a person standing and holding a book and to the side we can see a podium with mic and there is a laptop and some other objects on the table. we can see a person standing in the bottom right.
this pic. WER : 0.6041666666666666
this pic. BLEU: 0.5129596590369672
test number = 18 average, WER = 0.7047126293182373, BLEU = 0.4534094035625458

Decoder Only 2.2B
reference          : in this image we can see a person standing and holding a book and to the side we can see a podium with mic and there is a laptop and some other objects on the table. we can see a person standing in the bottom right.
hypothesis         : in this image we can see a woman standing holding a paper in there is a podium there is a laptop. in the background there can curtain.
bert corrected hypo: in this image we can see a woman standing holding a microphone in there is a podium there is a laptop. in the background there can be.
this pic. WER  : 0.6041666666666666
this pic. BLEU : 0.46518092088607155
this pic. WER2 : 0.6041666666666666
this pic. BLEU2: 0.46659631960687353
test number = 18 average, WER = 0.6953638195991516, WER2 = 0.6967058777809143, BLEU = 0.3822424113750458, BLEU2 = 0.37119778990745544
```
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2958180/4f6ad1e7-6036-47f2-b89e-4a32bdc9d0f2.png)

 
```
Decoder Only
hypo: in this image there see two woman on the face is the background .
refe: in this picture i can see 2 women in front and the women right is holding a brush in her hand and i see the paint on the face of the woman on the left and in the background i see the grass and on the top left of this image i see the blue color things.
this pic. WER : 0.8448275862068966
this pic. BLEU: 0.05620873821487133
test number = 19 average, WER = 0.7254354953765869, BLEU = 0.3494534194469452

Bert KD
hypo: in this image we can see two women is makeup face . in the background there is blurred .
refe: in this picture i can see 2 women in front and the women right is holding a brush in her hand and i see the paint on the face of the woman on the left and in the background i see the grass and on the top left of this image i see the blue color things.
this pic. WER : 0.8103448275862069
this pic. BLEU: 0.13240608597461212
test number = 19 average, WER = 0.7102722525596619, BLEU = 0.4365144968032837

Decoder Only 2.2B
reference          : in this picture i can see 2 women in front and the women right is holding a brush in her hand and i see the paint on the face of the woman on the left and in the background i see the grass and on the top left of this image i see the blue color things.
hypothesis         : in this image we can see two woman painting. in the background there can grass.
bert corrected hypo: in this image we can see two woman standing. in the background there can be.
this pic. WER  : 0.8275862068965517
this pic. BLEU : 0.1017159618593601
this pic. WER2 : 0.8448275862068966
this pic. BLEU2: 0.08600270019815176
test number = 19 average, WER = 0.7023229002952576, WER2 = 0.704501748085022, BLEU = 0.3674778640270233, BLEU2 = 0.35618752241134644

```
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2958180/8f3d3259-31e7-4723-9ad9-d4a240b9c968.png)

　

　
```
Decoder Only
hypo: in this image can see a polar water there is a there tree .
refe: in this image we can see an animal, water, rocks, and leaves. at the bottom of the image we can see a person who is truncated.
this pic. WER : 0.7419354838709677
this pic. BLEU: 0.2525684277567393
test number = 20 average, WER = 0.7262605428695679, BLEU = 0.34460917115211487

Bert KD
hypo: in this image we can see a polar in there is a water . in the there can see a tree .
refe: in this image we can see an animal, water, rocks, and leaves. at the bottom of the image we can see a person who is truncated.
this pic. WER : 0.6129032258064516
this pic. BLEU: 0.46931346785207634
test number = 20 average, WER = 0.7054038047790527, BLEU = 0.438154399394989

Decoder Only 2.2B
reference          : in this image we can see an animal, water, rocks, and leaves. at the bottom of the image we can see a person who is truncated.
hypothesis         : in this image we can a polar bear there can a person tree.
bert corrected hypo: in this image we can a polar bear there can a polar bear.
this pic. WER  : 0.7096774193548387
this pic. BLEU : 0.24872721074468593
this pic. WER2 : 0.7419354838709677
this pic. BLEU2: 0.2081390725229574
test number = 20 average, WER = 0.7026906609535217, WER2 = 0.7063735723495483, BLEU = 0.3615403175354004, BLEU2 = 0.3487851023674011
```
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2958180/d7423c3e-6202-4141-a8e8-11da69634d24.png)

　
　
```
Decoder Only
hypo: in this black and white image can see three persons standing and are on . in the background there can .
refe: in this image we can see three people, one of them is wearing a backpack, in front of them, we can see some bags, box, also we can see some plants, grass, and trees.
this pic. WER : 0.8571428571428571
this pic. BLEU: 0.37845386865337
test number = 21 average, WER = 0.7324930429458618, BLEU = 0.34622082114219666

Bert KD
hypo: in this is three black image there are three people ground . in the background there can trees .
refe: in this image we can see three people, one of them is wearing a backpack, in front of them, we can see some bags, box, also we can see some plants, grass, and trees.
this pic. WER : 0.8333333333333334
this pic. BLEU: 0.33130629577258996
test number = 21 average, WER = 0.7114956974983215, BLEU = 0.4330664277076721

Decoder Only 2.2B
reference          : in this image we can see three people, one of them is wearing a backpack, in front of them, we can see some bags, box, also we can see some plants, grass, and trees.
hypothesis         : in this image there can see three persons standing grass. in the background there can trees.
bert corrected hypo: in this image there can see three people standing together. in the background there can trees.
this pic. WER  : 0.7619047619047619
this pic. BLEU : 0.34963779902423403
this pic. WER2 : 0.7380952380952381
this pic. BLEU2: 0.35817260974682197
test number = 21 average, WER = 0.7055103778839111, WER2 = 0.7078840732574463, BLEU = 0.3609735369682312, BLEU2 = 0.34923213720321655
```

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2958180/3f145d18-638c-47a7-8f68-f55602a60612.png)
## Referemce

### AR system with CLIP + GPT2

https://github.com/toshiouchi/CLIP_textGPT2_v7_AR

### Mask prediction system with CLIP + BERT

https://github.com/toshiouchi/Img_Captioning_CLIP_BERT_Mask_Predict
