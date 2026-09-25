# Flask Sample App with Tests

This is a simple Flask web application with unit tests. The application provides a basic REST API for managing a list of items. It serves as a starting point for learning how to create a Flask application and write tests for it.

## Project Structure

The project is organized as follows:

- `app/`: Contains the Flask application and routes.
- `tests/`: Houses unit tests for the application.
- `run.py`: A script to run the Flask application.

## Getting Started

To get the Flask app up and running on your local machine, follow these steps:

1. **Clone the Repository:**

   ```bash
   git clone <repository_url>
   cd flask_sample_app
   ```

2. **Set Up a Virtual Environment:**

   It's recommended to create a virtual environment to isolate project dependencies.

   ```bash
   python -m venv venv
   source venv/bin/activate  # On Windows, use venv\Scripts\activate
   ```

3. **Install Dependencies:**

   Install the necessary dependencies using `pip`:

   ```bash
   pip install -r requirements.txt
   ```

4. **Run the Application:**

   Start the Flask application:

   ```bash
   python run.py
   ```

   The app will be available at [http://localhost:5000](http://localhost:5000).

5. **Run Tests:**

   To run the unit tests, execute the following command:

   ```bash
   python -m unittest discover tests
   ```

   This command will discover and run all tests in the `tests` directory.

## Application Routes

The application provides the following routes:

- `GET /`: Returns a simple greeting message.
- `GET /items`: Returns a list of items.
- `GET /items/{item_id}`: Returns the details of a specific item.
- `POST /items`: Adds a new item to the list.

## Testing

Unit tests are provided in the `tests` directory. They cover the basic functionality of the application, including route handling and response validation. You can use these tests as a reference to write your own tests or to verify the correctness of the application.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Contribute

Feel free to contribute to this project by opening issues or submitting pull requests. We welcome any improvements, bug fixes, or additional features.

## Author

- Pan Luo

## Acknowledgments

- This project was created as a sample Flask application for educational purposes.
- Special thanks to the Flask community for providing a fantastic web framework.

Enjoy experimenting with the Flask sample app! If you have any questions or need further assistance, please don't hesitate to reach out.
## Published Docker image

Public Docker Hub repository: https://hub.docker.com/r/henry178/msc-de1-flask-app

The tested image is `henry178/msc-de1-flask-app:1.0.0`.
The `latest` tag currently points to the same image.

To pull and run the published version:

```bash
docker pull henry178/msc-de1-flask-app:1.0.0
docker run --rm -p 5000:5000 henry178/msc-de1-flask-app:1.0.0 ```

## Docker Compose

From the repository root:

```cmd
docker compose up --build -d
docker compose ps
curl http://localhost:5000/
docker compose down
```

The container runs as UID 10001 and includes a health check.

## Local Kubernetes deployment

Requirements: Docker Desktop, kind and kubectl. The cluster configuration is in `kind/kind-config.yaml` and creates one control-plane node and two workers.

```cmd
kind create cluster --name msc-de1 --config kind\kind-config.yaml --wait 10m
kubectl get nodes --context kind-msc-de1
kubectl apply -f k8s\app.yaml --context kind-msc-de1
kubectl apply -f k8s\service.yaml --context kind-msc-de1
kubectl apply -f k8s\network-policy.yaml --context kind-msc-de1
kubectl rollout status deployment/flask-app -n msc-de1 --context kind-msc-de1
kubectl get pods,service -n msc-de1 --context kind-msc-de1
```

To access the API, keep this command running in one terminal:

```cmd
kubectl port-forward service/flask-app 5001:5000 -n msc-de1 --context kind-msc-de1
```

In another terminal:

```cmd
curl http://localhost:5001/
curl http://localhost:5001/items
```

The Deployment runs two replicas of the public Docker Hub image `henry178/msc-de1-flask-app:1.0.0`. It defines health probes, resource requests and limits, a non-root user and a read-only root filesystem.

## Kubernetes verification

Scaling from two replicas to three and back:

```cmd
kubectl scale deployment/flask-app --replicas=3 -n msc-de1 --context kind-msc-de1
kubectl get pods -n msc-de1 -o wide --context kind-msc-de1
kubectl scale deployment/flask-app --replicas=2 -n msc-de1 --context kind-msc-de1
```

Self-healing was verified by deleting one application Pod and observing that the Deployment created a replacement.

A rolling update was performed with image `henry178/msc-de1-flask-app:1.0.1`, followed by a rollback to `1.0.0`:

```cmd
kubectl set image deployment/flask-app flask-app=henry178/msc-de1-flask-app:1.0.1 -n msc-de1 --context kind-msc-de1
kubectl rollout status deployment/flask-app -n msc-de1 --context kind-msc-de1
kubectl rollout undo deployment/flask-app -n msc-de1 --context kind-msc-de1
kubectl rollout status deployment/flask-app -n msc-de1 --context kind-msc-de1
```

Version `1.0.1` was built with a version label to demonstrate the rollout; it does not change application functionality.

## Security and limitations

The Docker Scout scan and SPDX SBOM are available in `security/`. The scan is a snapshot of the image at the time of analysis; its findings should be reviewed before a production deployment.

The application's item list is held in process memory. Data is therefore not shared or persisted across Kubernetes replicas. The NetworkPolicy manifest is included, but network isolation requires a CNI that enforces NetworkPolicy; applying the manifest alone does not verify enforcement.