# EuroParl Results

| Model             | Params     | Pre-trained  | Fine-tuned | Tokenizer      | Decoding    | BLEU  |
|-------------------|------------|--------------|------------|----------------|-------------|-------|
| Transformer       | 19,960,216 | FALSE        | Europarl   | Word           | Greedy      | 1.39  |
| Transformer Align | 74,624,512 | EN2IT - OPUS | EN2RO - WMT 2016 | Sentence Piece | Greedy      | 11.89 |
| Transformer Align | 74,624,512 | EN2IT - OPUS | EN2RO - WMT 2016 | Sentence Piece | Beam Search | 12.25 |
| Transformer Align | 74,624,512 | EN2RO - OPUS | FALSE      | Sentence Piece | Greedy      | 43.83 |
| Transformer Align | 74,624,512 | EN2RO - OPUS | FALSE      | Sentence Piece | Beam Search | **43.84** |

# Weights: ![link](https://drive.google.com/drive/folders/1LoHHX3FV6f367NHQv1GfCeN4tucau48s?usp=sharing)

# Naive Approach

## Method:

**Pre-processing**: 
- Punctuation was removed;
- Text was converted to lowercase;
- [start] and [end] tags were added to Romanian sentences.

**Tokenizer**:
- Word-based;
- Vocabulary size is 15000;
- Sequence length was truncated to 20 words.

**Model**:
- Transformer;
- ~20 mil parameters.

**Training**:
- RMSprop;
- 40 epochs;
- Validation metric: accuracy.

**Model Diagram**:

![here](/assets/transformer.png)

**Convergence**:

![](/assets/1.png?raw=true)

## Results (on Europarl):

**Metric**:
- BLEU score (Greedy): 1.39;
- Accuracy (on the validation set): 0.5584.

**Examples**:
- “Madam President, Commissioner, ladies and gentlemen, I would like to thank all of the political groups and everyone who has spoken for the support and the constructive tone of their speeches." -> "doamnă preşedintă domnule comisar doamnelor şi domnilor aş dori să mulţumesc tuturor grupurilor politice şi tuturor celor care au [UNK]”;
- “It does not speak with one coherent voice, but with many voices." -> "nu este vorba de o voce clară [UNK] dar multe [UNK]”;
- “(The President cut off the speaker)" -> "preşedintele a întrerupto pe vorbitoare end”;
- “Stateless people are a separate issue and should be encouraged, using all means available, to apply for citizenship in their host country." -> "[UNK] oameni reprezintă o problemă și ar trebui [UNK] toate mijloacele disponibile pentru a aplica în vederea [UNK] în [UNK]”;
- “The debate is closed." -> "dezbaterea a fost închisă”.

**Limitations**:
- Many words were replaced with [UNK]. Even basic words were sometimes replaced when used in the plural form. This is partially attributed to the vocabulary size but more importantly, it’s a limitation of the tokenizer that we used. A good solution would be tokenizing sub-words such that grammar does not obstruct word lemmas;
- Sequence length was too short so some sentences were truncated (this can be partially avoided by running the model on multiple slices of the input sentence);
- The model doesn't output punctuations;
- Sometimes, sentences contain mistakes that are a direct result of our model’s limited learning capacity. Performance could be improved by using more recent architectures, pre-trained weights and bigger neural networks.

# Hugging Face - Helsinki (fine-tuned)

## Method:

**Pre-processing**: 
- Normalization.

**Tokenizer**:
- SentencePiece (trained on the OPUS dataset).

**Model**:
- Transformer Align;
- ~75 mil parameters;
- Pre-trained on English to Italian translation on the OPUS dataset.

**Training**:
- AdamW;
- 1 epoch;
- Validation metric: BLEU;
- Fine-tuned on English to Romanian translation on the WMT 2016 dataset.

**Model Diagram**: included in ![assets/](/assets/)

## Results (on Europarl):

**Metric**:
- BLEU score (Greedy): 11.89;
- BLEU score (Beam search): 12.25.

**Examples**:
- "Now, moreover, - and I take advantage of the fact that Mr Kallas is here - the Commission is placing in doubt the future central crossing of the Pyrenees in the future core network." -> "În plus, acum - şi profit de faptul că dl Kallas se află aici - Comisia pune sub semnul întrebării viitoarea traversare centrală a Pirineilor în viitoarea reţea centrală a Pirineilor în reţeaua de bază";
- "However, considering that we rarely revisit these types of issues as a House, it makes sense that we try and make any proposals on this as comprehensive as possible." -> "Cu toate acestea, având în vedere că rareori revizuim aceste tipuri de probleme în Parlament, este logic să încercăm să facem orice propuneri pe această temă cât mai cuprinzătoare posibil.";
 - "In this case, within two years, this programme will give us good arguments that we can use in the budget debates - a poker game involving the Commission, Parliament and the Member States - to obtain more money from the Member States in the interests of all the citizens of the European Union." ->
"În acest caz, în termen de doi ani, acest program ne va oferi argumente bune pe care le putem utiliza în dezbaterile privind bugetul - un joc de poker care implică Comisia, Parlamentul şi statele membre - pentru a obţine mai mulţi bani de la statele membre în interesul tuturor cetăţenilor Uniunii Europene."
- "I met a traveller from an antique land,<br />
Who said—“Two vast and trunkless legs of stone<br />
Stand in the desert. . . . Near them, on the sand,<br />
Half sunk a shattered visage lies, whose frown,<br />
And wrinkled lip, and sneer of cold command,<br />
Tell that its sculptor well those passions read<br />
Which yet survive, stamped on these lifeless things,<br />
The hand that mocked them, and the heart that fed;<br />
And on the pedestal, these words appear:<br />
My name is Ozymandias, King of Kings;<br />
Look on my Works, ye Mighty, and despair!."" -> "M-am întâlnit cu un călător dintr-un teren antic, Cine a spus-o, cu două picioare vastă şi calmă a staţiunii de piatră din deşert.. le-am apropiat, pe nisip, jumătate a scufundat o viză distrusă, a cărei bizară, pe bună dreptate, şi râpă de comandă rece, spunând că sculptorul său bine acei pasiuni citiţi încă pe care le supravieţuiesc, ştampilaţi pe aceste lucruri fără viaţă, mâna care i-a luat în derâdere şi inima care s-a alimentat; And pe patetal, aceste cuvinte".

**Limitations**:
- In the first example, the translation is syntactically correct, but wording is odd at the very end, because of the unnecessary repetition of "reţea". Increasing the repetition penalty of the generate method doesn't solve this issue;
- In the second example, the translation is almost perfect, which also yields a high BLEU score, but there is one semantic error of understanding that "House" means "Parlament". This kind of error is hard to spot for humans, because the terms are meronyms and the essential meaning is preserved;
- It makes translating errors on niche text such as poetry, which is also harder for humans, beacuse metaphors have multiple, subjective meanings. Poetry  syntax is also atypical. These problems could be solved by including more poetry translations in the training dataset;
- This model is severly undertrained because of computational restraints, but it's still an order of magnitude better than the first model, whose training converged.

# Hugging Face - Helsinki (pre-trained)

## Method:

The same architecture as above, this time directly using pre-trained weights from English to Romanian translation on the OPUS dataset.

## Results (on Europarl):


**Metric**:
- BLEU score (Greedy): 43.84;
- BLEU score (Beam search): 43.84.

**Examples**:
- "Now, moreover, - and I take advantage of the fact that Mr Kallas is here - the Commission is placing in doubt the future central crossing of the Pyrenees in the future core network." -> "Acum, în plus, - și profit de faptul că dl Kallas este aici - Comisia pune la îndoială viitoarea trecere centrală a Pirineilor în viitoarea rețea centrală..";
- "However, considering that we rarely revisit these types of issues as a House, it makes sense that we try and make any proposals on this as comprehensive as possible." -> "Cu toate acestea, având în vedere că rareori revizuim aceste tipuri de probleme în calitate de Parlament, este logic să încercăm să facem propuneri în acest sens cât mai cuprinzătoare posibil.";
- "In this case, within two years, this programme will give us good arguments that we can use in the budget debates - a poker game involving the Commission, Parliament and the Member States - to obtain more money from the Member States in the interests of all the citizens of the European Union." -> "În acest caz, în termen de doi ani, acest program ne va oferi argumente bune pe care le putem folosi în dezbaterile bugetare - un joc de poker care implică Comisia, Parlamentul și statele membre - pentru a obține mai mulți bani de la statele membre în interesul tuturor cetățenilor Uniunii Europene.";
- "I met a traveller from an antique land,<br />
Who said—“Two vast and trunkless legs of stone<br />
Stand in the desert. . . . Near them, on the sand,<br />
Half sunk a shattered visage lies, whose frown,<br />
And wrinkled lip, and sneer of cold command,<br />
Tell that its sculptor well those passions read<br />
Which yet survive, stamped on these lifeless things,<br />
The hand that mocked them, and the heart that fed;<br />
And on the pedestal, these words appear:<br />
My name is Ozymandias, King of Kings;<br />
Look on my Works, ye Mighty, and despair!." -> "Am întâlnit un călător dintr-un tărâm antic, care a spus două picioare vaste și fără trunchi de piatră Sta în deșert..... Lângă ei, pe nisip, jumătate scufundat un visage spulberat minciuni, a cărui încruntare, și buza încrețită, și smiorcăit de comandă rece, Spuneți că sculptorul ei bine acele pasiuni citite Care încă supraviețuiesc, ștampilate pe aceste lucruri lipsite de viață, Mâna care le-a batjocorit, și inima care a hrănit; Și pe piedestal, aceste cuvinte apar: Numele meu este Ozymandias, regele regilor; Uită-te la lucrările mele, voi Puternici, și disperare!".

**Limitations**:
- It handles EuroParl almost perfectly, as it is included in the OPUS dataset. The BLEU score is almost 4 times higher;
- In the second example, the same semantic error occurs;
- Niche text translation is improved, as the training set and the number of epochs is significantly increased. Though it is not perfected, and the same limitations occur;

# Resources:
https://keras.io/examples/nlp/neural_machine_translation_with_transformer/

https://medium.com/@tskumar1320/how-to-fine-tune-pre-trained-language-translation-model-3e8a6aace9f

https://arxiv.org/abs/1909.02074
