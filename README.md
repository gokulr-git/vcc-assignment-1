# vcc-assignment-1

Virtual Machines & Microservices Deployment



1. Introduction

This document provides a step-by-step guide for setting up multiple Virtual Machines (VMs) in VirtualBox, configuring their network, and deploying a microservice-based application across them. The microservices include a User Service, Product Service, and an API Gateway, each running on different VMs and communicating over a private network.

2. Virtual Machine Setup

2.1 Installation of VirtualBox

1. Download and install Oracle VirtualBox from [https://www.virtualbox.org](https://www.virtualbox.org).
2. Install the VirtualBox Extension Pack for enhanced features.

2.2 Creating Virtual Machines

1. Open VirtualBox and click on New.
2. Set the name (e.g., `VM1`, `VM2`, `VM3`), type as Linux, and version as Ubuntu (64-bit).
3. Allocate 2GB RAM and 10GB storage (dynamically allocated).
4. Choose the downloaded Ubuntu 24.10 ARM Server ISO and complete the installation.

2.3 Network Configuration

1. Go to Settings → Network for each VM.
2. Enable Adapter 1 as "NAT" for internet access.
3. Enable Adapter 2 as "Host-Only Adapter" (Attach to `vboxnet0`).
4. Start the VMs and check their IPs using:
   ```bash
   ip a
   ```
   Assign static IPs if necessary in `/etc/netplan/` YAML configuration.

3. Microservice Deployment

3.1 Install Dependencies on Each VM

Run the following on each VM:
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install python3 python3-pip -y
pip3 install flask requests
```








3.2 VM1: User Service

1. Create a directory and enter it:
   ```bash
   mkdir user_service && cd user_service
   ```
2. Create `app.py`:
   ```bash
   nano app.py
   ```
3. Add the following code:
   ```python
   from flask import Flask, jsonify
   app = Flask(__name__)
   
   @app.route('/users', methods=['GET'])
   def get_users():
       return jsonify({"users": ["Alice", "Bob", "Charlie"]})
   
   if __name__ == '__main__':
       app.run(host='0.0.0.0', port=5001)
   ```
4. Run the service:
   ```bash
   python3 app.py
   ```

3.3 VM2: Product Service

1. Create a directory:
   ```bash
   mkdir product_service && cd product_service
   ```
2. Create `app.py` and add:
   ```python
   from flask import Flask, jsonify
   app = Flask(__name__)
   
   @app.route('/products', methods=['GET'])
   def get_products():
       return jsonify({"products": ["Laptop", "Phone", "Tablet"]})
   
   if __name__ == '__main__':
       app.run(host='0.0.0.0', port=5002)
   ```
3. Run the service:
   ```bash
   python3 app.py
   ```










3.4 VM3: API Gateway

1. Create a directory:
   ```bash
   mkdir api_gateway && cd api_gateway
   ```
2. Create `app.py` and add:
   ```python
   from flask import Flask, jsonify, requests
   app = Flask(__name__)
   
   USER_SERVICE_URL = "http://192.168.56.101:5001/users"
   PRODUCT_SERVICE_URL = "http://192.168.56.102:5002/products"
   
   @app.route('/all-users', methods=['GET'])
   def all_users():
       response = requests.get(USER_SERVICE_URL)
       return jsonify(response.json())
   
   @app.route('/all-products', methods=['GET'])
   def all_products():
       response = requests.get(PRODUCT_SERVICE_URL)
       return jsonify(response.json())
   
   if __name__ == '__main__':
       app.run(host='0.0.0.0', port=5000)
   ```
3. Run the API Gateway:
   ```bash
   python3 app.py
   ```

4. Testing

Use `curl` from VM to test:
```bash
curl http://192.168.56.101:5001/users
curl http://192.168.56.102:5002/products
curl http://192.168.56.103:5000/all-users
curl http://192.168.56.103:5000/all-products
```
Expected JSON responses should be returned.



5.Conclusion

This document covered the installation, network setup, deployment, and testing of a microservice-based architecture using multiple Virtual Machines in VirtualBox.


