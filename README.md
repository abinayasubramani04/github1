
ABINAYA S ECE <abinaya.2202004@srec.ac.in>
5:10 PM (0 minutes ago)
to me

#include <ESP8266WiFi.h>
#include <ESP8266HTTPClient.h>

// Replace with your network credentials
const char* ssid = "Firmware";
const char* password = "Solutions@12345";

// Server URL
const char* serverUrl = "http://esskay-012024.live/agrimonitoring/save.php";

// Function to split a string by a delimiter and return the substring at the given index
String getValue(String data, char separator, int index) {
    int found = 0;
    int strIndex[] = { 0, -1 };
    int maxIndex = data.length() - 1;

    for (int i = 0; i <= maxIndex && found <= index; i++) {
        if (data.charAt(i) == separator || i == maxIndex) {
            found++;
            strIndex[0] = strIndex[1] + 1;
            strIndex[1] = (i == maxIndex) ? i + 1 : i;
        }
    }
    return (found > index) ? data.substring(strIndex[0], strIndex[1]) : "";
}

void setup() {
    Serial.begin(9600);
    WiFi.begin(ssid, password);

    // Wait for connection
    while (WiFi.status() != WL_CONNECTED) {
        delay(1000);
        Serial.println("Connecting to WiFi...");
    }

    Serial.println("Connected to WiFi");
}

void loop() {
    if (WiFi.status() == WL_CONNECTED) {
        if (Serial.available() > 0) {
            String myString = Serial.readString(); // Store UART serial data to myString variable
            myString.trim(); // Remove any leading or trailing whitespace

            // Split the string using the getValue function
            String val1 = getValue(myString, ',', 0); // Store first data
            String val2 = getValue(myString, ',', 1); // Store second data
            String val3 = getValue(myString, ',', 2); // Store third data
            String val4 = getValue(myString, ',', 3); // Store fourth data

            // Debugging: Print the extracted values
            Serial.println("Extracted Values:");
            Serial.println("val1: " + val1);
            Serial.println("val2: " + val2);
            Serial.println("val3: " + val3);
            Serial.println("val4: " + val4);

            // Ensure all values are extracted before sending
            if (val1.length() > 0 && val2.length() > 0 && val3.length() > 0 && val4.length() > 0) {
                HTTPClient http;
                http.begin(serverUrl);
                http.addHeader("Content-Type", "application/x-www-form-urlencoded");

                // Prepare POST data
                String postData = "temperature=" + val1 +
                                  "&humidity=" + val2 +
                                  "&pressure=" + val3 +
                                  "&altitude=" + val4;

                // Send POST request
                int httpCode = http.POST(postData);

                // Check the response
                if (httpCode > 0) {
                    String payload = http.getString();
                    Serial.println("Server response: " + payload);
                } else {
                    Serial.println("Error on HTTP request");
                }

                http.end();
            } else {
                Serial.println("Invalid data received. Skipping HTTP request.");
            }
        }
    } else {
        Serial.println("WiFi not connected");
    }

    // Delay as needed
    delay(2000);
}
