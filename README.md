# Semi-Supervised Learning Lipizzaner
This repository stores the code of **Lipi-SSL** project, a variant of **[Lipizzaner](https://github.com/ALFA-group/lipizzaner-gan)** to address **Semi-supervised Learning**. Lipizzaner is a framework to train generative adversarial networks (i.e., optimize their parameters) with gradient-based optimizers like Adam by using a distributed coevolutionary setup. 

The research work behind this project has been published in the following preprint paper:
- Jamal Toutouh, Subhash Nalluru, Erik Hemberg, & Una-May O'Reilly. (2023). **Semi-Supervised Learning with Coevolutionary Generative Adversarial Networks** [Preprint]. *Zenodo*, pp. 1--12 [https://doi.org/10.5281/zenodo.7840477](https://doi.org/10.5281/zenodo.7840477) 

### Experimental analysis

Lipi-SSL has been evaluated on the well-know image datasets MNIST and CIFAR-10. 

The configuration files used to run the experiments are the following ones:

- **General configuration file:** [`./src/configuration/quickstart-semi-supervised-learning/general.yml`](https://github.com/jamaltoutouh/lipi-ssl/blob/main/src/configuration/quickstart-semi-supervised-learning/general.yml)
- **MNIST instance:** [`./src/configuration/quickstart-semi-supervised-learning/mnist.yml`](https://github.com/jamaltoutouh/lipi-ssl/blob/main/src/configuration/quickstart-semi-supervised-learning/mnist.yml)
- **CIFAR-11 instance:** [`./src/configuration/quickstart-semi-supervised-learning/cifar.yml`](https://github.com/jamaltoutouh/lipi-ssl/blob/main/src/configuration/quickstart-semi-supervised-learning/cifar.yml)

These configuration files encapsulate the following Lipi-SSL hyperparamenter configuration:


| Parameter                         | MNIST                              | CIFAR-10                |
|-----------------------------------|------------------------------------|-------------------------|
| Network type                      | MNIST SSL-DCGAN                    | CIFAR SSL-DCGAN         |
| Input neurons to generator        | 100                                | 100                     |
| Discriminator hidden layers       | 4                                  | 7                       |
| Generator hidden layers           | 4                                  | 5                       |
| Neurons per hidden layer          | 16 384 - 131 072                   | 16 384 - 262 144        |
| Network complexity                | 128                                | 64                      |
| Generator output neurons          | 28x28                              | 64x64                   |
| Activation function               | $tanh$                             | $tanh$                  |
| Batch size                        | 300, 400, 600                      | 250, 200                |
| Label rates (in batch size order) | 0.01, 0.005, 0.00167               | 0.08, 0.1               |
| Stop condition                    | 100 training epochs                | 90 training epochs      |
| Tournament size                   | 2                                  | 2                       |
| Grid sizes                        | 1x1, 2x2, and 3x3.                 | 1x1 and 2x2             |
| Generator Performance measure     | FID                                | Inception Score (IS)    |
| Optimizer                         | Adam                               | Adam                    |
| Initial discriminator LR          | 0.001                              | 0.0007                  |
| Initial generator LR              | 0.0002                             | 0.0003                  |


### Lipi-SSL/Lipizzaner Setup

1. Install Docker
2. In the lipizzaner-gan folder run:
```
docker build -t lipizzaner-gan .
```
3. Link the files on your computer with the files in the container by running:
```
docker run -v "`pwd`":/lipizzaner-gan -ti --name lipizzaner-gan lipizzaner-gan
```

If the 3 steps outlined above are run successfully then you should be in the repo folder in the container.
```
root@925bdac2f9b3:/lipizzaner-gan/src#
```
### Dev environment setup
##### Pip
```
wget http://download.pytorch.org/whl/cu90/torch-0.3.1-cp36-cp36m-linux_x86_64.whl
```
```
pip3 install torch-0.3.1-cp36-cp36m-linux_x86_64.whl
```
```
pip3 install -r ./src/helper_files/requirements.txt
```
### Quick start

Lipizzaner includes a quick start example to test the installation on your localhost. In this experiment Lipizzaner trains a GAN by using four clients during 5 generations. The configuration files are located in `./src/configuration/quickstart/`. 

It is needed to set up some configuration parameters in `general.yml` to run the quick start experiment:
1. Set the clients IP addresses
2. Set the Losswise API key

Go to the `src` directory:
```
cd src
```

To run the quickstart experiments the four clients should be started (e.g. by runing the below commands):
```
python main.py train --distributed --client &
sleep 5;
python main.py train --distributed --client &
sleep 5;
python main.py train --distributed --client &
sleep 5;
python main.py train --distributed --client &
```

To start the master command line parameters are:
```
python main.py train --distributed --master -f configuration/quickstart/mnist.yml
```

### Notes

- The clients need to be restarted between non-sequential and sequential GAN training.

### *Distributed* training
Lipizzaner currently supports two types of training: distributed (in a master/client setup, with `AsynchronousEATrainer`) and local (with all other `*Trainer` classes).

To run Lipizzaner in a distributed way, you have two different options:
- Starting the client nodes manually. This requires to specify the nodes' addresses and ports in `general.yml` (note that autodetect is not recommended in this scenario).
- Using Docker to orchestrate the clients. Each client will run in a separate container of a docker swarm.

##### Option 1: Manual

1. Set the clients IP addresses in `general.yml`:
   - Make sure to use a square number of clients (e.g. 4, 9, ...)
   - Ports will be assigned automatically, starting at 5000 on each machine
   - **Examples**:
     - If you are using one machine with four clients, the port range will be 5000-5003
     - If you are using two machines with 8 clients each, the port ranges will be 5000-5007 and 5000-5007 on the respective machines
2. Start the client as often on each machine as you specified in `general.yml`
   - Command line parameters are `python main.py train --distributed --client`
3. Start the master, e.g. on your local machine
   - Command line parameters are `python main.py train --distributed --master -f configuration/lipizzaner-gan/celeba.yml`

The master will then wait until the clients are finished, collect the results, and terminate itself. The clients remain running, waiting for new experiments.


##### Option 2: Docker Swarms
1. Setup docker on all machines you want to use
2. Create a docker swarm and an overlay network with the following commands:

    *Execute on orchestrator (or 'manager') node:*
    ```
    docker swarm init --advertise-addr <YOUR_IP_ADDRESS>
    docker network create -d overlay lpz-overlay
    ```

    *Execute on worker nodes (the exact command, including the token, is returned by first of the above commands):*
    ```
    docker swarm join --token <TOKEN> <MANAGER_IP_ADDRESS>:2377
    ```

3. **Optional:** Login to a Docker Hub account that has access to the private lipizzaner2018/lipizzaner repository (with `docker login`) on each machine.
4. If you don't want to use Docker Hub, you alternatively can clone the repo and build the container on each machine.

    *To manually build the container, execute:*
    ```
    docker build -t lipizzaner2018/lipizzaner .
    ```
5. Run the Lipizzaner clients.

    *Execute on the **manager** node:*
    ```
    docker service create -e role=client --replicas <NR_OF_CLIENT> --network lpz-overlay --with-registry-auth --name lpz lipizzaner2018/lipizzaner:latest
    ```

    Docker will automatically distribute the clients over the all nodes in your swarm.

6. Run the Lipizzaner master to start the experiments.

    - **Make sure that autodiscover is set to `True` in `general.yml`, as this is required when running in docker.**

    - *Execute on any node:*
        ```
        docker run -it --rm -e config_file=CONFIG_FILE -e SWARM=True -e role=master --network lpz-overlay --name lipizzaner-master lipizzaner2018/lipizzaner:lates
        ```

        Set the config file path as you would for a non-docker run, e.g. `configuration/lipizzaner-gan/celeba.yml`.
        Lipizzaner wil automatically detect and use all non-busy nodes in the Docker overlay network.

7. When the experiment has finished, you can stop the client service (or keep it running for future experiments):
    ```
    docker service rm lpz
    ```


###### GPU support
GPU support for Docker Swarms is currently limited:
1. **[GPUs currently can't be shared](https://github.com/NVIDIA/nvidia-docker/issues/141#issuecomment-366911268) across multiple swarm services**
2. [nvidia-docker](https://github.com/NVIDIA/nvidia-docker) has to be installed on each machine
3. The docker daemon on each machine has to be [manually configured](https://github.com/NVIDIA/nvidia-docker/issues/141#issuecomment-356458450) to use the GPU


##### Option 3: Docker with GPU support
The most simple workaround for the limitations stated above (especially the first one) is to manually run the docker containers manually (not as swarm services):

1. Use the command described in the first point of the section above to create a swarm and an overlay network.
This is needed to establish communication between containers on multiple machines.

2. Login to Docker Hub (or manually build the image), also described above.

5. Run the Lipizzaner clients.

    *Execute the following command multiple times on each machine, e.g. 5 times each on 5 machines for 25 Lipizzaner clients:*

    ```
    docker run -d --rm -e role=client --runtime=nvidia --network lpz-overlay lipizzaner2018/lipizzaner:latest
    ```

6. Run the Lipizzaner master to start the experiments.

    - **Again, make sure that autodiscover is set to `True` in `general.yml`, as this is required when running in docker.**

    - *Execute on any node (notice that this command differs from the one in the previous section - '-e SWARM=True' was removed):*
        ```
        docker run -it --rm -e config_file=CONFIG_FILE -e role=master --network lpz-overlay --name lipizzaner-master lipizzaner2018/lipizzaner:lates
        ```

        Set the config file path as you would for a non-docker run, e.g. `configuration/lipizzaner-gan/celeba.yml`.
        Lipizzaner wil automatically detect and use all non-busy nodes in the Docker overlay network.

Some local implementations are parallelized, others are not (as the focus was on the distributed setup).

### Generating samples from a mixture
After a distributed training session finished, the results will be gathered on the master node (saved to `<output>/<trainer>/master`).
These results contain both sample data and the resulting neural network models in PyTorch's .pkl format.

To generate sample data from such a mixture, use the following command:

```
python main.py generate --mixture-source <SOURCE_DIR> -o <TARGET_DIR> --sample-size 100 -f <CONFIG_FILE>
```

Make sure that `<SOURCE_DIR>` points to a directory that contains both the .pkl files and the mixture.yml configuration file (created by Lipizzaner after training). Use the same `<CONFIG_FILE>` as for the training process. A concrete example could look like this:


```
python main.py generate --mixture-source ./output/lipizzaner_gan/master/2018-06-04_10-01-50/128.30.103.19-5000 -o ./output/samples --sample-size 100 -f configuration/lipizzaner-gan/celeba.yml
```

### Working With Sequential Data

#### Creating network traffic dataset

In order to create a `pcap` file of the network traffic from your local machine, navigate to `lipizzaner-gan/src/data/network_data` and use the following command:

`sudo ./collect_network_traffic.sh`

And let this run for sufficiently long to create a large enough pcap file. Note that you may need to change the name of the `pcap` file specified in the `bash` file in order to generate multiple datasets.

#### Extracting netflow information

`argus` and `argus-client` is required for this step. In order to convert this `pcap` into a `numpy` file with the desired fields pass `--pcap_file` to `analyze_network_file.py` script with the pcap file you're creating the dataset from, and run the following command:
```
python analyze_network_file.py --pcap_file ${PCAP_FILE}
```
This will create a `.npy` file in the same directory, with the name you specified in the `file_to_analyze` line. In order to use this numpy file in the `NetworkDataLoader` class, update the `flow_data` line with the directory to the generated `.npy` file.

#### Training on Network Traffic Data

To run lipizzaner on a sequential dataset of network traffic flows, start multiple clients with the command from the /src directory:

`python main.py train --distributed --client`

Then, start a master process with the following command:

`python main.py train --distributed --master -f configuration/lipizzaner-gan/network_traffic.yml`

### Further guidelines
If you want to add your own data to Lipizzaner, refer to [this tutorial](docs/howto/add-dataloader-to-lipizzaner.md).
For guidelines about how to compile and run the Lipizzaner dashboard, check its [README](src/lipizzaner-dashboard/README.md).
