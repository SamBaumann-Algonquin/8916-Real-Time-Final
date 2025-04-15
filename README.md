# CST8916_Final_Project_Assignment

## Scenario Description:

For CST8916's final project, we'll use our knowledge of real-time systems to devise a simulation of a system in place to sample various points of data about the Ottawa Rideau Canal. Gathering accurate data every few seconds is a daunting task for any human to collect, record and transmit, so we can instead create scripts to automate this process. These scripts can generate and contiually measure fictional temperature, ice thickness and snow accumulation, and process this information coming from three seperate locations, serving as a baseline for how such a solution could be implemented in the real world.

**Proposed Solution**

3 sensors are placed in 3 different locations around the canal (Dow's Lake, Fifth Avenue, and NAC).
Using Microsoft Azure's IoT Hub, we can process real-time data of weather and ice data from these sensors.
All the data will then be examined by Azure Stream Analytics every 5 minutes, and subsequently stored in Azure Blob Storage so as to accurately determine if the condition of the canal is safe enough to be open to the public in a timely and cost-efficient manner.

## System Architecture:
![Rideau example diagram](/screenshots/simple_dataflow_sim.png)

## Implementation Details:

**IoT Sensor Simulation**

To simulate real-time ice monitoring, we can three Python scripts, each of which acts as a virtual IoT device for a specific location:

- dow_lake_simulator.py

- fifth_ave_simulator.py

- nac_simulator.py

Each script will continually generate random values for:

- Ice Thickness (cm)
- Surface Temperature (°C)
- Snow Accumulation (cm)
- External Temperature (°C)
- Timestamp (ISO UTC format)

This data is formatted in JSON and sent to the Azure IoT Hub using the Azure IoT Device SDK:

**Sample JSON Payload:**

    {
        "location": "Dow's Lake",
        "iceThickness": random.randint(20, 35),
        "surfaceTemperature": random.randint(-10, 1),
        "snowAccumulation": random.randint(0, 15),
        "externalTemperature": random.randint(-15, 4),
        "timestamp": datetime.utcnow().isoformat() + "Z"
    }
Each script uses a device-specific connection string, and pushes data every 10 seconds.

**Azure IoT Hub Configuration**

From the Microsoft Azure portal, we can create an IoT hub to manage our project. We can create an IoT hub with three devices, one for each location, and pair them with our scripts. These are then linked to Azure's Stream Analytics tool, which will process the JSON payloads and send them to an Azure Blob storage account for longer term processing. See the screenshots folder for additional details on configuring Azure IoT, Analytics and Blob Storage!

## Azure Blob Storage
Processed data is saved in an Azure Storage account:.

**Folder Structure**

    skateway/YYYY/MM/DD/HH/
    
**File Format:** JSON 

**Example Blob Path:**

    skateway/2025/04/14/17/
**Sample Output JSON:**

    {
      "location": "Dow's Lake",
      "windowEnd": "2025-04-09T17:40:00Z",
      "avgIceThickness": 27.2,
      "maxSnowAccumulation": 14
    }

To aggregate this data, we can set a rule for the following query to be run on our database every 5 minutes:

    SELECT
        location,
        System.Timestamp AS windowEnd,
        AVG(CAST(iceThickness AS float)) AS avgIceThickness,
        MAX(CAST(snowAccumulation AS float)) AS maxSnowAccumulation
    INTO
        [processed-data-blobOutput]
    FROM
        [RideauSkatewayHubInput] TIMESTAMP BY timestamp
    GROUP BY
        TumblingWindow(minute, 5), location

This will ensure our data is as fresh as reasonably possible, and the collection rate can be increased or decreased as users see fit.

## Usage Instructions:

**Running the IoT Sensor Simulation**

First, make sure that Python is installed on your device, and install the azure-iot-device dependencies:

    pip install azure-iot-device

Navigate to the sensor-simulation/ directory in your project folder.

Open three seperate command prompts, one for each script.

Run each script using:

    python dow_lake.py
    python fifth_ave.py
    python nac.py

The scripts will send live information (every 10 seconds) to the Azure IoT Hub using each device’s unique connection string.

**Configuring Azure Services**

**IoT Hub Setup**

These steps assume you have access to a Microsoft Azure subscription.

- Navigate to your Azure Portal

- Search for and create an IoT Hub. Once deployed, navigate to "Devices" and create three new devices, one for each script. Copy and paste their unique connection strings into the appropriate scripts (more details visible in the screenshots folder!)

**Stream Analytics Job Setup**

Create a Stream Analytics Job

Under Inputs, add a new input of type IoT Hub. Give it an alias, and indicate that it will collect from all devices in JSON format.

Under Outputs, add an output of type Blob Storage. Also give this an alias, and link it to a blob storage account under a resource group of your choice (if you do not have a blob storage account, create one first - check the screenshots!) Note that it's format will be JSON, and line separated. Here, we use a Path pattern of skateway/{date}/{time}

Under Query, paste the following (identical to the query above):

    SELECT
        location,
        System.Timestamp AS windowEnd,
        AVG(CAST(iceThickness AS float)) AS avgIceThickness,
        MAX(CAST(snowAccumulation AS float)) AS maxSnowAccumulation
    INTO
        [processed-data-blobOutput]
    FROM
        [RideauSkatewayHubInput] TIMESTAMP BY timestamp
    GROUP BY
        TumblingWindow(minute, 5), location

- Click Save Query, then Start Job > Choose Now > Start

**Accessing Stored Data in Blob Storage**

- Navigate to your Azure Storage Account

- Open the Containers section and locate your blob storage container

- Navigate the folders based on the date and hour:

        skateway/2025/04/14/17/

- Open or download the output files (JSON or CSV)

  - Each file contains results grouped by location and time window

  - Use tools like Notepad, VS Code, or Excel to view the data

The output files will look something like this!

    {
      "location": "Dow's Lake",
      "windowEnd": "2025-04-09T17:45:00Z",
      "avgIceThickness": 27.6,
      "maxSnowAccumulation": 12
    }

## Results:

Once linked, results will be continually generated! Note that:

- Data is grouped by location

- Trends across time are captured

- Information is ready for future use in dashboards, alerts, or decision-making tools

## Conclusion

And now we've completed a functional simulator for measuring critical data about the Rideau Canal! Some challenges that a user may face when performing these steps are:

- Ensuring Python has access to your computer's PATH variable
- Ensuring three seperate terminals are each up and running a different python script for information generation
- Accurately matching each identified string to its corresponding python script

If these issues persist, be sure to check the screenshots folder and follow along as closely as possible to avoid any ambiguity or errors!
