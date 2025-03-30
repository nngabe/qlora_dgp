# LoRA conditioned Deep Gaussian Processes 
This repository contains code for training deep gaussian processes for VaR prediction conditioned on LLM embeddings, e.g. LLaMa-3 finetuned using LoRA/QLoRA. Finetuning data is retrieved using the newsdata API.  Combining finetuned LLMs and deep GPs is a good choice when:

(1) There is reasonably large and up to date corpus for training and generating RAG embeddings;

(2) The number of assets under consideration is relatively small, i.e. <img src="https://latex.codecogs.com/svg.image?\large&space;&space;N<100" />, since GP inference scales as <img src="https://latex.codecogs.com/svg.image?\large&space;&space;\mathcal{O}(N^3)" />;

(3) Robust uncertainty estimates are crucial for success of the application (e.g. risk estimation with VaR).

## Background

Value at Risk (VaR) is a statistical measure that estimates the maximum potential loss an investment portfolio might experience, within a defined timeframe and confidence level, assuming stationary market conditions. VaR can be optimized based on a portfolio with weights <img src="https://latex.codecogs.com/svg.image?\large&space;&space;w_i" /> and assets <img src="https://latex.codecogs.com/svg.image?\large&space;&space;f_i" /> represented as deep gaussian processes:

<img src="https://latex.codecogs.com/svg.image?\LARGE&space;\begin{matrix}f&=\sum_iw_if_i\\f_i&\sim\textrm{GP}(\mu_i,K)\end{matrix}" />

where the kernel <img src="https://latex.codecogs.com/svg.image?\large&space;&space;K" /> is parameterized by a neural network <img src="https://latex.codecogs.com/svg.image?\large&space;&space;\phi:\mathbb{R}^n\rightarrow\mathbb{R}^m" /> [[1]](https://arxiv.org/pdf/1511.02222)

<img src="https://latex.codecogs.com/svg.image?\large&space;&space;\begin{matrix}K_{ij}=k\big(\phi(\mathbf{x}_i),\phi(\mathbf{x}_j)\big)\end{matrix}" />

and the kernel function is, e.g., an RBF kernel

<img src="https://latex.codecogs.com/svg.image?\large&space;&space;k(\mathbf{x}_i,\mathbf{x}_j)=\textrm{exp}\Big(-\frac{1}{2}||\mathbf{x}_i-\mathbf{x}_j||/l^2\Big)" />.

Since GPs produce probabilistic outputs, we can evaluate the VaR based on a trained GP and a user-specified risk level ($\alpha$ = 0 is risk-free, $\alpha$ = 1 is maximal risk) [[2]](https://arxiv.org/pdf/2105.06126) 


<img src="https://latex.codecogs.com/svg.latex?\Large&space;V_{\alpha}(\mathbf{x},\mathbf{z})=\textrm{inf}\{\omega:P(f(\mathbf{x},\mathbf{z})\leq\omega)\geq\alpha\}" title="\Large V_{\alpha}(\mathbf{x},\mathbf{z})=\textrm{inf}\{\omega:P(f((\mathbf{x},\mathbf{z}))\leq\omega)\geq\alpha\}" />

where $f(\mathbf{x},\mathbf{z})$ is a deep GP conditioned on a context embedding $\mathbf{z}$ generated by our finetuned LLaMa model. The VaR is defined as the minimum upper bound $\omega$ required such that $f(\mathbf{x},\mathbf{z})$ is less than the specified risk level.

## How to Use This Repository

1. First install the requirements:
   ```
   pip install -r requirements.txt
   ```
   
3. Collect text from the newsdata API by running
   ```
   python grab_news_text.py
   ```
   The text retrieved can be filtered by country, category, or language with the params argument.
   
5. Preprocess and filter raw text data using
   ```
   python fineweb-2-pipeline.py --dataset="path/to/your/dataset"
   ```
   
7. Finetune with
   ```
   tune run lora_finetune_single_device --config ./8B_lora_newstext.yaml
   ```
   
9. Train deep GP with
    ```
   python dgp.py
    ```
