## Install TensorFlow on Apple M2 (M1, Pro, Max) with GPU (Metal)

**Note:** Uninstall Anaconda/Anaconda Navigator and other related previously installed version of conda-based installations. Anaconda and Miniforge cannot co-exist together.

Installing Miniforge
--------------------

*   Install [miniforge](https://github.com/conda-forge/miniforge) from [brew](https://formulae.brew.sh/cask/miniforge): `brew install miniforge`
*   Create an anaconda environment: `conda create -n tf`
*   Activate the environment: `conda activate tf`
*   Install Python: `conda install python`

Installing TensorFlow
---------------------

*   Run: `conda install -c apple tensorflow-deps` to install Apple's TensorFlow dependencies
*   Run: `pip install tensorflow-metal` to install Apple's Metal GPU APIs for TensorFlow
*   Execute: `pip install tensorflow-macos` to install MacOS arm64 version of TensorFlow
*   Execute: `pip install tensorflow-datasets pandas jupyterlab` to install relevant dependencies to run sample code.

Verifying Installation
----------------------

*   Execute: `jupyter-lab` to open a Jupyter Notebook and run the following code:

```python
    import tensorflow as tf
    import tensorflow_datasets as tfds
    print("TensorFlow version:", tf.__version__)
    print("Num GPUs Available: ", len(tf.config.experimental.list_physical_devices('GPU')))
    print("Num CPUs Available: ", len(tf.config.experimental.list_physical_devices('CPU')))
```

Running A Sample Code (MNIST)
-----------------------------
```python
    (ds_train, ds_test), ds_info = tfds.load(
        'mnist',
        split=['train', 'test'],
        shuffle_files=True,
        as_supervised=True,
        with_info=True,
    )
    
    def normalize_img(image, label):
      """Normalizes images: `uint8` -> `float32`."""
      return tf.cast(image, tf.float32) / 255., label
    
    batch_size = 128
    ds_train = ds_train.map(
        normalize_img, num_parallel_calls=tf.data.experimental.AUTOTUNE)
    ds_train = ds_train.cache()
    ds_train = ds_train.shuffle(ds_info.splits['train'].num_examples)
    ds_train = ds_train.batch(batch_size)
    ds_train = ds_train.prefetch(tf.data.experimental.AUTOTUNE)
    ds_test = ds_test.map(
        normalize_img, num_parallel_calls=tf.data.experimental.AUTOTUNE)
    
    ds_test = ds_test.batch(batch_size)
    ds_test = ds_test.cache()
    ds_test = ds_test.prefetch(tf.data.experimental.AUTOTUNE)
    
    model = tf.keras.models.Sequential([
      tf.keras.layers.Conv2D(32, kernel_size=(3, 3),
                     activation='relu'),
      tf.keras.layers.Conv2D(64, kernel_size=(3, 3),
                     activation='relu'),
      tf.keras.layers.MaxPooling2D(pool_size=(2, 2)),
      tf.keras.layers.Flatten(),
      tf.keras.layers.Dense(128, activation='relu'),
      tf.keras.layers.Dense(10, activation='softmax')
    ])
    
    model.compile(
        loss='sparse_categorical_crossentropy',
        optimizer=tf.keras.optimizers.Adam(0.001),
        metrics=['accuracy'],
    )
    
    model.fit(
        ds_train,
        epochs=12,
        validation_data=ds_test,
    )
```

Credits
-------

*   [Nachiket Sirsikar](https://www.linkedin.com/in/nachiketsirsikar/)

[Previous issue](https://sudhanva-narayana.ghost.io/install-kubeflow-on-digital-ocean-machine-learning/)

[Browse all issues](https://sudhanva-narayana.ghost.io/page/2)

[Next issue](https://sudhanva-narayana.ghost.io/install-pytorch-on-apple-m1-m1-pro-max-gpu/)