# Conditional Adversarial Camera Model Anonymization

PyTorch implementation of [Conditional Adversarial Camera Model Anonymization](https://arxiv.org/abs/2002.07798) (ECCV 2020 Advances in Image Manipulation Workshop).

Digital photographs can be **blindly** attributed to the specific camera model used for capture.

<p align="center"><img src="images/blind-att.png"  /></p>

Conditional Adversarial Camera Model Anonymization (Cama) offers a way to preserve privacy by transforming these artifacts such that the apparent capture model is changed (targeted transformation). That is, given an image and a target label condition, the applied transformation causes a non-interactive black-box *target* (i.e. to be attacked/fooled) convnet classifier <img src="https://render.githubusercontent.com/render/math?math=\large F"> to predict the target label given the transformed image. While at the same time retaining the original image content.

However, Cama is trained in a **non-interactive black-box setting**: Cama does not have knowledge of the parameters, architecture or training randomness of <img src="https://render.githubusercontent.com/render/math?math=\large F">, nor can Cama interact with it. 

<p align="center"><img src="images/cam-anon.png" /></p>

## Anonymizing in-distribution images
Cama is able to successfully perform targeted transformations on in-distribution images (i.e. images captured by camera models known to it).

Example (below) of Cama transformed images <img src="https://render.githubusercontent.com/render/math?math=\large x^'"> with different target label conditions <img src="https://render.githubusercontent.com/render/math?math=\large y^'"> given an in-distribution input image <img src="https://render.githubusercontent.com/render/math?math=\large x"> (whose ground truth label is <img src="https://render.githubusercontent.com/render/math?math=\large y">). The applied transformations (amplified for visualization purposes) are shown as <img src="https://render.githubusercontent.com/render/math?math=\large \delta">.

<p align="center"><img src="images/flower.png" width="700" /></p>

## Anonymizing out-of-distribution images
Cama is also able to successfully perform targeted transformations on out-of-distribution images (i.e. images captured by camera models unknown to it).

Example (below) of Cama transformed images <img src="https://render.githubusercontent.com/render/math?math=\large x^'"> with different target label conditions <img src="https://render.githubusercontent.com/render/math?math=\large y^'"> given an out-of-distribution input image <img src="https://render.githubusercontent.com/render/math?math=\large x"> (whose ground truth label is <img src="https://render.githubusercontent.com/render/math?math=\large y">). The applied transformations (amplified for visualization purposes) are shown as <img src="https://render.githubusercontent.com/render/math?math=\large \delta">.

<p align="center"><img src="images/building.png" /></p>

## Model
We denote by <img src="https://render.githubusercontent.com/render/math?math=\large x\in\mathbb{R}^d"> and <img src="https://render.githubusercontent.com/render/math?math=\large y\in\mathbb{N}_c=\{1,\dots,c\}"> an image and its ground truth (source) camera model label, respectively, sampled from a dataset <img src="https://render.githubusercontent.com/render/math?math=\large p_{\text{data}}">. Consider a *target* (i.e. to be attacked) convnet classifier <img src="https://render.githubusercontent.com/render/math?math=\large F"> with <img src="https://render.githubusercontent.com/render/math?math=\large c"> classes trained over input-output tuples <img src="https://render.githubusercontent.com/render/math?math=\large (x,y)\sim p_{\mathrm{data}}(x,y)">. Given <img src="https://render.githubusercontent.com/render/math?math=\large x">, <img src="https://render.githubusercontent.com/render/math?math=\large F"> outputs a prediction vector of class probabilities <img src="https://render.githubusercontent.com/render/math?math=\large F:x\mapsto F(x)\in[0,1]^{c}">.

As aforementioned, Cama operates in a **non-interactive black-box setting**. We do, however, assume that Cama can sample from a dataset similar to <img src="https://render.githubusercontent.com/render/math?math=\large p_{\mathrm{data}}">, which we denote by <img src="https://render.githubusercontent.com/render/math?math=\large q_{\mathrm{data}}">. Precisely, Cama can sample tuples of the following form: <img src="https://render.githubusercontent.com/render/math?math=\large (x,y)\sim q_{\text{data}}(x,y)"> s.t. <img src="https://render.githubusercontent.com/render/math?math=\large y\in\mathbb{N}_{c^'}">, where <img src="https://render.githubusercontent.com/render/math?math=\large (x,y)\sim q_{\text{data}}(x,y)"> s.t. <img src="https://render.githubusercontent.com/render/math?math=\large c^' \leq c">. That is, the set of possible image class labels in <img src="https://render.githubusercontent.com/render/math?math=\large p_{\text{data}}"> is a superset of the set of possible image class labels in <img src="https://render.githubusercontent.com/render/math?math=\large q_{\text{data}}">, i.e. <img src="https://render.githubusercontent.com/render/math?math=\large \mathbb{N}_{c}\supseteq \mathbb{N}_{c'}">.

Suppose <img src="https://render.githubusercontent.com/render/math?math=\large (x,y)\sim q_{\text{data}}(x,y)"> and <img src="https://render.githubusercontent.com/render/math?math=\large y^' \in\mathbb{N}_{c^'}">, where <img src="https://render.githubusercontent.com/render/math?math=\large y^' \neq y"> is a target label. Our aim is to learn a function <img src="https://render.githubusercontent.com/render/math?math=\large G:(x,y^')\mapsto x^' \approx x"> s.t. the maximum probability satisfies <img src="https://render.githubusercontent.com/render/math?math=\large \argmax_{i} F(x^')_i=y^'">. This is known as a *targeted* attack, whereas the maximum probability of an *untargeted* attack must satisfy <img src="https://render.githubusercontent.com/render/math?math=\large \arg \max_{i} F(x^')_i\neq y">. **Cama performs targeted attacks.**

<p align="center"><img src="images/model.png" /></p>

Cama has two class conditional components: a generator <img src="https://render.githubusercontent.com/render/math?math=\large G"> that transforms an image <img src="https://render.githubusercontent.com/render/math?math=\large x"> conditioned on a target class label <img src="https://render.githubusercontent.com/render/math?math=\large y^'">, and a discriminator <img src="https://render.githubusercontent.com/render/math?math=\large D"> that predicts whether the low-level high-frequency pixel value dependency features of any given image conditioned on a label are real or fake. In addition, Cama has a fixed (w.r.t. its parameters) dual-stream discriminative decision-making component <img src="https://render.githubusercontent.com/render/math?math=\large E"> (evaluator) that decides whether a transformed image <img src="https://render.githubusercontent.com/render/math?math=\large x"> belongs to its target class <img src="https://render.githubusercontent.com/render/math?math=\large y">. In essence, <img src="https://render.githubusercontent.com/render/math?math=\large E"> serves as a surrogate for the non-interactive black-box <img src="https://render.githubusercontent.com/render/math?math=\large F">. W.r.t. <img src="https://render.githubusercontent.com/render/math?math=\large E">, a transformed image <img src="https://render.githubusercontent.com/render/math?math=\large x^'"> is decomposed into its high and low spatial frequency components (<img src="https://render.githubusercontent.com/render/math?math=\large x^'_\text{H}"> and <img src="https://render.githubusercontent.com/render/math?math=\large x^'_\text{L}">, respectively), via <img src="https://render.githubusercontent.com/render/math?math=\large E_0">, with each assigned to a separate stream (<img src="https://render.githubusercontent.com/render/math?math=\large E_\text{H}"> and <img src="https://render.githubusercontent.com/render/math?math=\large E_\text{L}">, respectively). The evaluator then reasons over the information present in <img src="https://render.githubusercontent.com/render/math?math=\large x^'_\text{H}"> and <img src="https://render.githubusercontent.com/render/math?math=\large x^'_\text{L}"> separately (via <img src="https://render.githubusercontent.com/render/math?math=\large E_\text{H}"> and <img src="https://render.githubusercontent.com/render/math?math=\large E_\text{L}">, respectively). This reinforces the transformation process, as <img src="https://render.githubusercontent.com/render/math?math=\large G"> is constrained to transform both high and low spatial frequency camera model-specific artifacts used by the evaluator for discrimination.


During conditional adversarial training, <img src="https://render.githubusercontent.com/render/math?math=\large G"> minimizes:

<p align="center"><img src="images/g-minimizes.png" width="700" /></p>

whereas <img src="https://render.githubusercontent.com/render/math?math=\large D"> minimizes:

<p align="center"><img src="images/d-minimizes.png" width="700" /></p>

### Previous approaches to camera model anonymization
<p align="center"><img src="images/previous-works.png" /></p>
