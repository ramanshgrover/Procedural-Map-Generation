# Procedural-Map-Generation
Procedural Fantasy Map Generation using DCGAN trained in conjunction with a pix2pix GAN.

## Table of Contents
  1. [Motivation](#Motivation)
  2. [Prerequisites and Dependencies](#Prerequisites-and-Dependencies)
  3. [Data Acquisition](#Data-Acquisition)
  4. [Methodology](#Methodology)

## Motivation
In Computer Graphics and Virtual Environment Development, a large portion of time is devoted to creating assets – maps being one of them – as they usually form the basis of these large graphical worlds. Traditionally, the “procedural” generation of maps is accomplished by employing smartly designed (but handcrafted) algorithms that can create impressive texture renders that look visually stunning, however, can appear somewhat simple and repetitive due to mathematical constraints and hence, require human intervention. 

So far, maps have been procedurally generated through a host of algorithms designed to mimic real-life cartography. Given the advent of Deep Learning, it would only seem like a relevant and exciting endeavour to leverage the power of generative networks (such as Generative Adversarial Networks ([Goodfellow et al., 2014](https://arxiv.org/pdf/1406.2661.pdf))) to learn algorithms to automatically generate maps, without the need to manually write algorithms to do so. Therefore, We aim to create a  fantasy map generator utilising GANs to the most of their potential for effectively automating the generation process (without human involvement) and developing non-algorithmic map generators.
 
## Prerequisites and Dependencies
Ensure that you have [Python 3.5+](https://www.python.org/downloads/) and [pip3](https://pip.pypa.io/en/stable/installing/#installing-with-get-pip-py) installed on your specific OS distribution.

Clone the repository and create a virtual environment.
```shell
git clone https://github.com/ramanshgrover/Procedural-Map-Generation/
python -m venv .env
source .env/bin/activate
```

Install [dependencies](https://github.com/ramanshgrover/Procedural-Map-Generation/tree/master/requirements.txt) directly by
```shell
cd Procedural-Map-Generation/
pip3 install -r requirements.txt
``` 
**Disclaimer:** _This may take a while._
 
## Data Acquisition
For the purpose of this project, we decided to utilise heightmap data generated from a Procedural Map Generator as the scope of this exploratory experiment is to examine the capabilities of Deep Generative Networks when contrasted with existing Procedural Map Generators. Moreover, it seemed more appropriate as there already exists literature with variations of (real-life) high resolution NASA heightmap and terrain data. We opted for Azgaar’s [Fantasy Map Generator](https://github.com/Azgaar/Fantasy-Map-Generator) which is based and inspired off of Martin O'Leary's [Generating Fantasy Maps](http://mewo2.com/notes/terrain/), Amit Patel's [Polygonal Map Generation for Games](www-cs-students.stanford.edu/~amitp/game-programming/polygon-map-generation), and Scott Turner's [Here Dragons Abound](https://heredragonsabound.blogspot.com/) and provides a wide array of customisable options. We generated and fetched about 6000 simple height maps and their equivalent texture maps (as adding rivers, provinces, and other terrain parameters complicated the problem) with an automated [fetch script](https://github.com/ramanshgrover/Procedural-Map-Generation/blob/master/Data/fetch.ipynb). As expected of noise-based algorithmic procedural map generators, we observed similar recurrent iterations of maps generated after every few epochs which contributes to the specificity and limitation of the dataset used. Some sample datapoints can be seen below and the generated dataset can be found [here](https://drive.google.com/drive/folders/1kR7Ilj2McrhY4wTYyZALRhkcKYpiDvCe?usp=sharing). Download and Extract it into the `Data/Maps/` directory.

![Dataset](https://github.com/ramanshgrover/Procedural-Map-Generation/blob/master/Images/dataset.png)

## Methodology
### Experimentation
As a first step, we train a DCGAN which maps samples from the prior z'~p(z) to a generated heightmap x' = G<sub>h</sub>(z'). While we experienced some issues with training stability we were able to generate heightmaps that were somewhat faithful to the original images. Apart from the aforementioned stability issues, the generated heightmaps also exhibited the presence of small-scaled artifacts which we tackled by adding a slight blur to the final images via a Gaussian kernel convolution. This served to smooth out the weird artifacts generated by the DCGAN G<sub>h</sub> and smoothened out the heightmaps for visualization (as discussed in the next section). A pix2pix GAN G<sub>t</sub> was jointly trained in conjunction to synthesize the equivalent texture map.

### Executing the code
After the dataset is downloaded, execute [export.ipynb](https://github.com/ramanshgrover/Procedural-Map-Generation/blob/master/Data/export.ipynb) to generate the HDF5 file. 

To run an experiment, you need to specify a particular experiment. These are defined in `experiments.py`. We recommend you use the experiment called [`test1_nobn_bilin_both`](https://github.com/ramanshgrover/Procedural-Map-Generation/blob/master/experiments.py#L113-L125). The `experiments.py` file is what you run to launch experiments, and it expects two command-line arguments in the form `<experiment name> <mode>`. For example, to run `test1_nobn_bilin_both` we simply do:

```shell
python experiments.py test1_nobn_bilin_both train
```
(**NOTE:** you will need to modify this file and change the URL to point to where your .h5 file is.)

For all the Theano bells and whistles, you can execute `run.sh`

Various things will be dumped into the results folder (for the aforementioned experiment, this is `output/test1_nobn_bilin_both`) when it is training. The files you can expect to see are:
* `results.txt`: various metrics in .csv format that you can plot to examine training.
* `gen_dcgan.png,disc_dcgan.png`: architecture diagrams for the DCGAN generator and discriminator.
* `gen_p2p.png,disc_p2p.png`: architecture diagrams for the P2P generator and discriminator.
* `out_*.png`: outputs from the P2P GAN, showing the predicted Y from ground truth X.
* `dump_a/*.png`: outputs from the DCGAN, in which an X's are synthesised from z's drawn from the prior p(z).
* `dump_train/*.a.png,dump_train/*.b.png`: high-res ground truth X's along with their predicted Y's for the training set.
* `dump_valid/*.a.png,dump_valid/*.b.png`: high-res ground truth X's along with their predicted Y's for the validation set.

### Visualisation
The maps generated in the previous part are then processed and 3D visualized in this step. First, the height map and the texture map are obtained. Both the maps are then converted to grayscale using the average grayscale formula:

where, g<sub>r</sub> = (r+g+b)/3
	
g<sub>r</sub> is the value of red, green and blue for new grey pixel.
r, g, and b are the respective values of red, green and blue in the original pixel.
 
From these grey pixel values, height (intensity) values are obtained by scaling them to get a value between 0 and 1. As mentioned in the previous section, a slight blur is applied with a Gaussian Kernel Convolution on the final heightmaps to accommodate for any sharp / rigid edges and irregularities, making the final render smoother and continuous. We use the height values obtained from the grayscale image as the  coordinate to plot the 3D map. Finally, the respective texture mesh is placed over the 3D Visualization and the final 3D map is then saved as an HTML file for viewing. You can check out a sample visualization within our [Data Visualisation Notebook](https://github.com/ramanshgrover/Procedural-Map-Generation/blob/master/Data%20Visualization.ipynb)

![visualisation](https://github.com/ramanshgrover/Procedural-Map-Generation/blob/master/Images/smooth.png)

### Future Work
 * Better Handcrafted Dataset
 * Using Spacial GAN over DCGAN
 * A splatmap segmentation pipeline
