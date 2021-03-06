# t5-mcq-generation
Generate MCQ Using T5 Transformer

# Downloading the models
This project uses T5 trained model to generate question from any context and BERT trained model to generate the options. The requried models are not available in github as the max size per file is restricted to 100 MB. The models can hence be downloaded from shared google drive's link

### 1. T5 Trained Model with key
https://drive.google.com/file/d/1_WRScNZ3m_Frmh8z401NRZx7VkDvosOf/view?usp=sharing

### 2. T5 Trained Model without key
https://drive.google.com/file/d/1-1YqKnDkQbAHKrvuxYG1xy2dHmDxbNVN/view?usp=sharing

### 3. BERT trained model from distractor generation
https://drive.google.com/drive/folders/10Krkv8G0vYGShsu6J3dOav9ja301auM8?usp=sharing

Before trying to generate MCQ place the models in your google drive. 

# Generating a Question
For generating question generators/QuestionGenerator.ipynb file can be used. Below are the steps to execute the file
1. Use google colab to open the file
2. Restart runtime once simpletransformer is installed
3. Provide the location of the tar ball using cell 3 <pre><code>!tar -zxvf /content/drive/MyDrive/Models/model_with_key.tar.gz</code></pre>
4. Once the model is extracted it can be found at location <pre><code>t5_model_directory = "outputs/best_model</pre></code>Change the location if default location does not works.
5. Provide a new sentence using context variable and key using answer variable. The predicted question will be stored in mcq variable.

<b>Note:</b> GPU is required to generate questions

```python
context = 'John\'s enjoys playing cricket in summer'
answer = 'cricket'
generator = QuestionGenerator(context, answer)
mcq = generator.get_mcq()
```

# Generating Distractors
Distractors are the wrong answers for multiple-choice question. For generating distractors DistractorGenerator.ipynb file can be used. Below are the steps to execute the file.
1. Use google colab to open the file
2. Download the BERT-WSD model from the link provided above and upload it to your google drive.
3. Change the model directory to the path were model is stored in the drive.
4. provided a sentence and a target word, the distractor generator class will generate distractors. Following are the steps to execute

```python
sentence = "John enjoys playing cricket in summer"
targetword = "cricket"
distractorGenerator = DistractorGeneration(sentence, targetword)
formattedSentence = distractorGenerator.format_sentence_for_bert()
sense,meaning,answer = distractorGenerator.get_predictions(formattedSentence)
distractors = distractorGenerator.generate_distractors_wordnet(sense, answer)
```
<b>Output: </b> 

            John enjoys playing [TGT]cricket[TGT] in summer

            meaning:  a game played with a ball and bat by two teams of 11 players; teams take turns trying to score runs 
            score:  0.9997342482658158

            meaning:  leaping insect; male makes chirping noises by rubbing the forewings together 
            score:  0.0002657517341842314


            -----------------------------------------------------------------------------------------------

            ['ball_game', 'field_hockey', 'football', 'hurling', 'lacrosse', 'polo', 'pushball', 'ultimate_frisbee']

# Training Scripts
The scripts used for training T5 models are located in train directory.

# Text Summarization and Answer generation using T5-Base
The text summarization and sample answer prediction script can be done using generators/text_summarization.ipynb.
```python
# Summarization
text = """
The National Union of Freedom Fighters (NUFF) was an armed Marxist revolutionary group in Trinidad and Tobago. The group fought a guerrilla campaign to overthrow the government of Prime Minister Eric Williams following the failed 1970 Black Power uprising and a mutiny in the Trinidad and Tobago Regiment. NUFF formed from the Western United Liberation Front, a loose grouping of largely unemployed men from the western suburbs of Port of Spain. NUFF drew disaffected members of the National Joint Action Committee, a Black Power organisation, and established a training camp in south Trinidad. In 1972 and 1973 NUFF attacked police posts to acquire weapons, robbed banks, and carried out an insurgent campaign against the government. With improved intelligence capabilities, the government eventually killed or captured most of its leadership. Eighteen NUFF members and three policemen were killed over the course of the insurgency. NUFF was anti-imperialist and anti-capitalist and was notable for the extent to which women played an active role in the organisation, including among its guerrilla fighters.
"""
c_string = ComplexString(text)
c_string.summary()
```

<b>Output: </b> the national union of freedom fighters was an armed revolutionary group. it fought a guerrilla campaign to overthrow the government of prime minister Eric Williams. NUFF attacked police posts to acquire weapons, robbed banks, carried out insurgent campaign.

```python
# Prediction of answer given a context and question
context = 'the national union of freedom fighters was an armed revolutionary group. it fought a guerrilla campaign to overthrow the government of prime minister Eric Williams. NUFF attacked police posts to acquire weapons, robbed banks, carried out insurgent campaign.'
question = "Who is the prime minister"
input = 'question: %s context: %s' % (question, context)
features = tokenizer(input, return_tensors='pt')
input_ids = features['input_ids']
attention_mask = features['attention_mask']
outcome = model.generate(input_ids = input_ids, attention_mask = attention_mask, max_length=255)
tokenizer.decode(outcome[0])
```
<b>Output: </b>Eric Williams

# Text complexity Analysis
Text complexity can be found using the script file generators/text_simplification.ipynb
```python
text = 'the national union of freedom fighters was an armed revolutionary group. it fought a guerrilla campaign to overthrow the government of prime minister Eric Williams. NUFF attacked police posts to acquire weapons, robbed banks, carried out insurgent campaign.'
import textstat
textstat.flesch_kincaid_grade(text)
```
<b>Output: </b> 10.7

# Evaluation
The t5-base model is already trained for answering the question given context and question as input. To evaluate the performance of our model we generated question using our trained model and passed the question along with the context to t5-base model. The answer generated by t5-base is compared with the input key for equality. The evaluation script is present inside evaluation folder. The below code shows a sample evaluation. Due to hardware limitation we have run the evaluation on 100 records only
```python
class Evaluate():
  def __init__(self):
    self.ques_generator = QuestionGenerator()
    self.ans_generator = AnswerGenerator()

  def eval(self):
    correct = 0
    for record in validation_list[:100]:
      context = record['context']
      key = record['answer']
      question = self.ques_generator.get_mcq(context, key)
      answer = self.ans_generator.get_answer_for_question(question, context)
      if key == answer:
        correct += 1
    return correct

correct_preds = Evaluate().eval()
```
<b>Output: </b>34
