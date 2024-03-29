---
title: "Install PyTorch on Apple M2 (M1, Pro, Max) with GPU"
datePublished: Mon May 23 2022 01:36:46 GMT+0000 (Coordinated Universal Time)
cuid: cl7amaoy902tmaznvgh4w8vzb
slug: install-pytorch-on-apple-m1-m2-pro-max-gpu
canonical: https://sudhanva.me/install-pytorch-on-apple-m1-m1-pro-max-gpu/
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1661526318586/u1o3KaFI2.jpeg
tags: machine-learning, deep-learning, installation, pytorch

---

Note:
-----

*   **Uninstall** Anaconda/Anaconda Navigator and other related previously installed version of conda-based installations.
*   Anaconda and Miniforge **cannot co-exist** together.
*   However, if you have previous installations of PyTorch with Miniforge, you can continue to use that **without uninstalling** it.
*   We will install the GPU version in a new `conda` environment anyway.
*   Keep in mind that the GPU version is still a **nightly build** and expect it to have **breaking changes** for the next couple of months.

Installing Miniforge
--------------------

*   Install [miniforge](https://github.com/conda-forge/miniforge) from [brew](https://formulae.brew.sh/cask/miniforge): `brew install miniforge`
*   Create an anaconda environment: `conda create -n pt122`
*   Activate the environment: `conda activate pt122`
*   Install Python: `conda install python`

Installing PyTorch
------------------

*   Run: `pip install --pre torch torchvision torchaudio --extra-index-url https://download.pytorch.org/whl/nightly/cpu` to install PyTorch (Nightly) dependencies
*   Execute: `pip install pandas jupyterlab` to install relevant dependencies to run sample code.
*   Run: `conda activate pt122`

Verifying Installation
----------------------

*   Execute: `jupyter-lab` to open a Jupyter Notebook and run the following code:

    import torch
    torch.device("mps")
    torch.__version__
    torch.tensor([1,2,3], device="mps")

Credits
-------

*   [Nachiket Sirsikar](https://www.linkedin.com/in/nachiketsirsikar/)
