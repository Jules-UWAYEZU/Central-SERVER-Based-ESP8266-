# Central-SERVER-Based-ESP8266-
 ESP8266 based Central system to receive and process data from multiple NODEs and deploy on Web dashboard
 #include <ESP8266WiFi.h>
#include <ESP8266WebServer.h>

const char* ssid = "TEMP_AP";
const char* password = "12345678";

ESP8266WebServer server(80);

float temperature = 0;

// -------------------- DATA UPDATE ENDPOINT --------------------
void handleData() {
  if (server.hasArg("temp")) {
    temperature = server.arg("temp").toFloat();
    Serial.println("Temp updated: " + String(temperature));
  }
  server.send(200, "text/plain", "OK");
}

// -------------------- JSON API FOR LIVE DATA --------------------
void handleTemp() {
  String json = "{";
  json += "\"temperature\":";
  json += String(temperature);
  json += "}";

  server.send(200, "application/json", json);
}

// -------------------- DASHBOARD UI --------------------
void handleRoot() {
  String html = R"rawliteral(
<!DOCTYPE html>
<html>
<head>
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>IoT Temperature Dashboard</title>

  <style>
    body {
      margin: 0;
      font-family: Arial, sans-serif;
      background: #ffffff;
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
      color: #111827;
    }

    .card {
      width: 300px;
      padding: 25px;
      border-radius: 18px;
      border: 2px solid #22c55e;
      box-shadow: 0 10px 25px rgba(34, 197, 94, 0.15);
      text-align: center;
    }

    h2 {
      margin: 0;
      color: #16a34a;
      font-size: 20px;
    }

    .temp {
      font-size: 48px;
      font-weight: bold;
      margin: 20px 0;
      color: #15803d;
    }

    .status {
      display: inline-block;
      padding: 6px 12px;
      background: #dcfce7;
      color: #166534;
      border-radius: 999px;
      font-size: 12px;
      font-weight: bold;
    }

    .label {
      font-size: 13px;
      color: #6b7280;
      margin-top: 10px;
    }
  </style>
</head>

<body>

  <div class="card">
    <h2>🌡 Temperature Monitor</h2>

    <div class="label">Current Reading</div>
    <div class="temp" id="temp">-- °C</div>

    <div class="status">LIVE</div>
  </div>

  <script>
    function updateData() {
      fetch('/temp')
        .then(response => response.json())
        .then(data => {
          document.getElementById("temp").innerHTML =
            data.temperature.toFixed(1) + " °C";
        });
    }

    setInterval(updateData, 2000);
    updateData();
  </script>

</body>
</html>
)rawliteral";

  server.send(200, "text/html", html);
}

// -------------------- SETUP --------------------
void setup() {
  Serial.begin(115200);

  WiFi.softAP(ssid, password);
  Serial.println("Access Point Started");
  Serial.println(WiFi.softAPIP());

  server.on("/", handleRoot);
  server.on("/data", handleData);
  server.on("/temp", handleTemp);

  server.begin();
}

// -------------------- LOOP --------------------
void loop() {
  server.handleClient();
}
