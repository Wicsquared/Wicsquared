from flask import Flask, request, jsonify
import json

app = Flask(__name__)

# Default V2Ray configuration template
def generate_v2ray_config(uuid, port, network="ws", host="example.com"):
    return {
        "inbounds": [
            {
                "port": int(port),
                "protocol": "vmess",
                "settings": {
                    "clients": [
                        {"id": uuid, "alterId": 64}
                    ]
                },
                "streamSettings": {
                    "network": network,
                    "wsSettings": {"headers": {"Host": host}}
                }
            }
        ],
        "outbounds": [
            {"protocol": "freedom", "settings": {}}
        ]
    }

@app.route("/generate", methods=["GET"])
def generate_config():
    uuid = request.args.get("uuid", "default-uuid")
    port = request.args.get("port", 8080)
    network = request.args.get("network", "ws")
    host = request.args.get("host", "example.com")
    
    config = generate_v2ray_config(uuid, port, network, host)
    return jsonify(config)

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000, debug=True)
