# Slides:
https://docs.google.com/presentation/d/1EMDO9gOJ_zo-1ZIe-Os1tx40hJTVLY9uYBZjlvNIJSs/edit?usp=sharing

# Tutorial:
# https://tech-habit.info/posts/https-cert-based-auth-with-flask-and-gunicorn/

# Certificates:
# WARNING!!! 
# Common Name MUST be different for each certificate !!!

openssl req -nodes -new -x509 -days 365 -keyout ca.key -out ca-crt.pem

# create server private key and server CSR
openssl req -nodes -new -keyout server.key -out server.csr

# generate certicate based on server's CSR using CA root certificate and CA private key
openssl x509 -req -days 365 -in server.csr -CA ca-crt.pem -CAkey ca.key -CAcreateserial -out server.crt

# verify the certificate (optionally)
openssl verify -CAfile ca-crt.pem server.crt



# create client private key and client CSR
openssl req -nodes -new -keyout client.key -out client.csr

# generate certicate based on client's CSR using CA root certificate and CA private key
openssl x509 -req -days 365 -in client.csr -CA ca-crt.pem -CAkey ca.key -CAcreateserial -out client.crt

# verify the certificate (optionally)
openssl verify -CAfile ca-crt.pem client.crt


# Code (minimal) :
from flask import Flask
import ssl


app = Flask(__name__)


@app.route('/ping')
def ping():
    return 'pong'


if __name__ == '__main__':
    context = ssl.SSLContext(ssl.PROTOCOL_TLSv1_2)
    context.verify_mode = ssl.CERT_REQUIRED
    context.load_verify_locations('ca-crt.pem')
    context.load_cert_chain('server.crt', 'server.key')
    app.run('0.0.0.0', 8080, ssl_context=context)


# run python script and then test
# curl --insecure --cacert ca-crt.pem --key client.key --cert client.crt https://localhost:8080/ping

# reply should be pong
