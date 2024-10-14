# Category Theory Paper Generator

Category theory is a branch of mathematics known for its abstract and often incomprehensible appearance, leading some to refer to it humorously as "abstract nonsense." Inspired by its abstraction, this project aims to generate humorous, nonsensical papers that at a glance resemble serious category theory research. The generation process is powered by the bigcode/starcoderbase-1b model—a language model pre-trained on 80+ programming languages—fine-tuned on LaTeX documents sourced from arXiv, all using the Hugging Face platform.

Explore the generated papers in the fake_paper_pdf folder. For a particularly entertaining read, I recommend checking out fake_math_paper006.tex, titled "Solving Every Open Problem in Math With Category Theory." The paper explores deep mathematical insights, frequently referencing "the concept of iterated iterated iterated recursion"—truly cutting-edge stuff!

## Dataset

The code for gathering data can be found in scrape.ipynb. Using the arXiv API, I collected source files for around 1,000 recent publications in category theory. While I cannot upload the dataset due to arXiv’s Terms of Service, I focused solely on LaTeX documents that begin with \documentclass. Each file averages 2,000 lines of LaTeX code, resulting in an estimated dataset size of approximately 2 million lines of LaTeX.

## Generating Papers

Everything described below is implemented in Fake_Paper_Generator.ipynb. 

### Data Preprocessing

- **Tokenization and Chunking:** Since transformer models like bigcode work with fixed-length input sequences, I created a class called ConstantLengthDataset. This class converts raw LaTeX text into tokenized inputs, ensuring each chunk fits within size limits by buffering text and calculating token count estimates based on a subset of the dataset.
- **Fill-in-the-Middle (FIM):** I applied the Fill-in-the-Middle approach to a portion of the data. FIM trains the model to fill in missing sections of LaTeX code, not just generate left-to-right. This improves the model’s ability to handle structured documents, like theorems and proofs, that are common in mathematical papers.

### Fine-Tuning and Inference

- **Model Fine-Tuning:** The fine-tuning process leverages LoRA (Low-Rank Adaptation), which reduces the number of trainable parameters from 1.14 billion to just 5.55 million. This makes training more efficient and achievable on a single A100 GPU, with one epoch taking approximately two hours.

- **Text Generation:** To generate coherent multi-page papers, I carefully tweaked the inference parameters (temperature, top-k, and top-p sampling). Papers are generated in chunks of 256 tokens at a time, where the model's output is fed back into the next generation step. In the final iteration, a Fill-in-the-Middle suffix of \end{document} is added to ensure the LaTeX can be compiled into a proper PDF.
