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

```python
context = 'John\'s enjoys playing cricket in summer'
answer = 'cricket'
generator = QuestionGenerator(context, answer)
mcq = generator.get_mcq()
```

# Generating Distractors
