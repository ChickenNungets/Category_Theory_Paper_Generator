# Category Theory Paper Generator

Category Theory is a branch of mathematics that is famous(or infamous) for its abstract and incomprehensible appearence, with some calling it 'abstract nonsense'. I figured with the subject being so abstract it would be fun to generate humorous and nonsensical papers that at a glance have the appearance of category theory research. The generation process is powered by a bigcode/starcoderbase-1b(a model pretrained on 80+ programming languages) model fine-tuned on LaTeX documents sourced from arXiv. This is all done via higgingface. You can check out the generated papers in the fake_paper_pdf folder - A particularly enlightening paper I reccomend is fake_math_paper006. The prompt I used to generate this paper has the title 'Solving Every Open Problem In Math With Category Theory'. Our clever model builds off this prompt and repeatedly employs "the concept of *iterated iterated
iterated recursion* ."

## Dataset

The code for data gathering can be found in scrape.ipnb. I used the arxiv API to gather source files for around 1000 recent publications in category theory. In compliance with the Arxiv TOS I will not upload the dataset. I extract and keep only the latex documents that begin with `\documentclass` - with each file having around 2000 lines of latex code, I would estimate that the dataset in total contains around 2 million lines of latex code. 

## Generating Papers

Everything described below is implemented in Fake_Paper_Generator.ipynb. 

### Data Preprocessing

- Since transformer models like bigcode work with fixed-length input sequences, I implemented a class called ConstantLengthDataset, which is an iterable dataset that returns constant-length chunks of tokens. It reads a buffer of text from the original dataset until it hits the size limits, then applying the tokenizer to convert the raw text into tokenized inputs. It uses an estimation of tokens per character computed on a subset of the data.
- The Fill-in-the-Middle (FIM) approach was applied to a subset of the data, allowing the model to train not only on sequential left-to-right generation but also in contexts where the middle of a LaTeX document is missing, enhancing the model's ability to handle structured text like mathematical papers.

### Bigcode Fine-tuning and Inference

For fine tuning I use LoRA (Low-Rank Adaptation of Large Language Models), which brought the number of trainable parameters from 1,142,761,472 to 5,554,176. This allowed training for one epoch to run on a single A100 GPU in about two hours.

Generating somewhat coherent papers required a little tweaking of the inference parameters (temperature, top k, top p, etc.). To generate multi page papers, I generate parts 256 tokens at a time, feeding the output back as input for new text generation, adding a FIM suffix of `\end{document}` at the last iteration to ensure the result could be compiled.

