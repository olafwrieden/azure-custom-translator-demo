# Azure Custom Translator - Demo

**Disclaimer**: This content is not officially endorsed by *Microsoft*.

## My Project Vision

### Background

Teo Reo Māori is the indigenous language of New Zealand and formally recognised as one of New Zealand's official languages since 1987.

Interestingly, Māori is the first language worldwide to be modelled in Microsoft Translator using Neural Machine Translation (NMT), over traditional models. This presents an exciting opportunity for both private and public sector organisations with domain-specific terminology to build their very own Custom Translator (CT) models on Microsoft Azure.

### This Project

Upon completion of this project, parties interested in experimenting with the deployment of CT shall be able to:

1. Understand the prerequisites for training, testing, and tuning their CT models.
2. Deploy, connect, and interact with this demo application.
3. *Stretch Goal:* Click-to-Deploy directly into Azure via Azure Resource Manager (auto-provisioning).

## Initial Architectural Design Thoughts

Assuming the model is trained, a user uploads a text file to the application which uploads it to Blob Storage (may help with processing of parallel data), an Event Grid trigger calls an Azure Function to pipe the texts through Custom Translator, upon completion, an HTTP trigger on an Azure Function saves the translated output (could include confidence score) into Blob Storage and returns the data on screen or by file download to the user.

### Early Learnings

We need a minimum of 10,000 training sentence pairs to create a model.

- Azure's Cognitive Services include Translator Text, extensive documentation for which, can be found [here](https://docs.microsoft.com/en-us/azure/cognitive-services/translator/).

### Creating a Custom Translator Project

We begin by following [this guide](https://docs.microsoft.com/en-us/azure/cognitive-services/translator/custom-translator/quickstart-build-deploy-custom-model) to:

1. Create a Workspace
2. Create a Project
3. Upload training, tuning and testing document sets
4. Create a Model
5. Analyse our Model
6. Deploy our Model

### Understanding Input Data

- When uploading text, we want to ensure the text is UCS-2 LE BOM encoded to force 16-bit characters, not 8. This will take care of macrons (a line above a vowel to indicate that it should be spoken as a long vowel), such as: ā, ē, ī, ō, ū, Ā, Ē, Ī, Ō and Ū.
- To upload multiple sets of documents of the same type (e.g. all Training data), we can create a ZIP file containing parallel data, to be parsed by Custom Translator.
- All files are strictly limited to 100MB (maximum file size).
- When using single and double quotes for speech punctuation, ensure that they are balanced, else the model is biased towards one.
- If our language has a well-established Microsoft Translator model publicly available, and we are missing one side of the translation in our data set, one idea is to feed it through the existing translator and filling the gap in our input with this data with the currently available "baseline" output. We would want to ensure samples of these translations are reviewed for structure and wording by native speakers regularly.

#### Training

Training data is a combination of every type of sentence. A minimum of 10,000 language pairs are required. From this count, 2,500 pairs are extracted for testing and tuning.
Like all data types, we can upload parallel or archive data files and prefer `.tmx` or `.xlsx` formats. However, the issue with individual xlsx files is the difficulty of maintaining and parsing files with more than 10,000 or 100,000 lines of text. For this reason, the parallel data upload is better suited to keep translations aligned, especially when data is in the millions.

We also have an opportunity to work with `.align` file types, which allow us to tell Custom Translator that no further alignment of sentences with their respective translation is necessary.

#### Testing

Knowing that many organisations don't build their own testing data sets, Custom Translator removes 2,500 sentence pairs from the training data set to use as testing data. The reason they are removed entirely from the training data is to avoid 1-to-1 sentence matches, which occur when the given testing sentence is also a member of the training set.

Testing data tests our model's accuracy and should be a good representation of what a customer might be translating (length, syntax etc). With Neural Machine Translation, words our model has not come across before are copied across as-is.

#### Tuning

Tuning data can be used during testing and isn't touched until later in the process when we want to check how our model is doing from an error perspective (Training - Testing = Difference). Each testing data set is only run once, to prevent it from leaking into our training set.

#### Phrase Dictionary

A Phrase Dictionary is used for the translation of compound nouns and sentences which don't change over time. When a sentence is parsed, we check if a substring exists in a phrase dictionary and replace it before sending it through the translation engine. An example phrase might be: "Māori Language Commission" = "Te Taura Whiri i te Reo Māori".
This "find and replace" is case sensitive!

#### Sentence Dictionary

Similar to the Phrase Dictionary, a sentence dictionary can between 500 to 2500 entries in length. It is not case sensitive! Between sentence and phrase dictionaries, we "find and replace" sentences first, then phrases, then it feeds through the translation engine and is joined up again.

We can use the phrase and sentence dictionaries to refine our translations after testing, however, it is better to improve the quality of the input data (training data) than to fix it afterwards.