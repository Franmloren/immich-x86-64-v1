As Immich updated to version 3.0.0, old devices that use x86-64-v1 architecture stopped working with the immich-machine-learning. In this repository I'll post the custom image that I've made using immich 3.0.3 for fix that problem and the step by step of how to make it in any immich version. This happens because in immich version 3 the program started using a newer version of NumPy and ONNX Runtime that no longer support the older hardware. The solution is to change the version of this engines used by the app.

What you will need:

A server running immich with docker

A newer linux or WSL environment

1. Clone the immich repo at your local newer enviroment:

git clone --branch v3.0.3 https://github.com/immich-app/immich.git

2. Go to the machine-learning directory:

cd immich/machine-learning

3. Open the .toml file:

nano pyproject.toml

4. Change the values:

from numpy>=2.4.0,<3.0",

to numpy==2.3.5",


from cpu = ["onnxruntime>=1.23.2,<2"]

to cpu = ["onnxruntime==1.22.0"]


5. Install uv to build your docker image:

curl -LsSf https://astral.sh/uv/install.sh | sh

6. Make sure uv is using the variables needed for full support in your image:

uv python install 3.11

uv python pin 3.11

uv add "numpy==2.3.5"

uv add "onnxruntime==1.24.1"

uv add "insightface==0.7.3"

7. Start the uv environment:

uv lock

uv sync

8. Make sure the environment updated correctly the variables:

uv run python -c "import numpy; print(numpy.__version__)"

uv run python -c "import onnxruntime; print(onnxruntime.__version__)"

uv run python -c "import insightface; print(insightface.__version__)"

The output should be:

2.3.5

1.24.1

0.7.3

9. Build the image:

docker build \

  --build-arg DEVICE=cpu \
  
  -t immich-machine-learning-custom .

10. Now you must check the image:

docker run --rm \

  immich-machine-learning-custom \
  
  python -c "import numpy,onnxruntime,insightface; print(numpy.__version__, onnxruntime.__version__, insightface.__version__)"

11. Confirme that the image was created:

docker images

you must see something like 

immich-machine-learning-custom latest    xxxxxxxx

12. Export your docker image

docker save -o immich-machine-learning-custom.tar immich-machine-learning-custom:latest

13. Send your image to the server using scp:

*Do not paste on the root directory of your server, try something like /home/user/

scp immich-machine-learning-custom.tar user@server-ip:/path/destination/

14. Log in to your server using ssh

ssh user@server-ip

15. Go to your file path using cd and confirm its there using ls

16. Load your image to docker:

docker load -i immich-machine-learning-custom.tar

17. Confirm that the image was loaded:

docker images

18. Shutdown your immich

19. Navigate to immich docker-compose.yml file path:

*this is in my case using casaos, search to see where yours should be

cd /var/lib/casaos/apps/immich

20. Edit the .yml file:

sudo nano docker-compose.yml

from image: ghcr.io/immich-app/immich-machine-learning:v3.0.3

to image: immich-machine-learning-custom:latest

21. Start your immich-machine-learning with the new image:

docker compose up -d immich-machine-learning

22. Start the rest of your immich server

23. DONE!
