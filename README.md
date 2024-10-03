# Fronius Datamanager Logger and Monitor

This repository contains a Docker Compose stack designed to capture real-time photovoltaic (PV) production data from a Fronius inverter with Datamanager. The stack consists of two services:

- **Telegraf**: An agent that listens for push calls from the inverter through an exposed HTTP endpoint.
- **InfluxDB**: A database that stores the PV production data for analysis and monitoring.

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Cloning the Repository](#cloning-the-repository)
3. [Configuration](#configuration)
   - [.env File](#env-file)
   - [InfluxDB Setup](#influxdb-setup)
   - [Restart Telegraf Service](#restart-telegraf-service)
   - [Fronius Inverter Configuration](#fronius-inverter-configuration)
4. [Running the Stack](#running-the-stack)
5. [Port Configuration](#port-configuration)

## Prerequisites

Before starting, ensure you have the following installed:

- Docker
- Docker Compose

You will also need access to your Fronius inverter's web interface for configuring the push service.

## Cloning the Repository

Start by cloning this repository to your local machine:

```bash
git clone https://github.com/juaniten/fronius-datamanager-monitor.git
cd fronius-pv-data-logger
```

## Configuration

### .env File

You need to create a `.env` file from the provided `.env.example` template to configure the environment variables.

1. Copy the `.env.example` file:

   ```bash
   cp .env.example .env
   ```

2. Edit the `.env` file and configure the following:
   - `DOCKER_INFLUXDB_INIT_ORG`: Name of your organization for InfluxDB.
   - `DOCKER_INFLUXDB_INIT_BUCKET`: The bucket where the data will be stored.
   - `TELEGRAF_MEASUREMENT_NAME_OVERRIDE`: Measurement name for Fronius push data.
   - `TELEGRAF_ENDPOINT_PATH_NAME`: Endpoint path that Telegraf will expose. This should match the Fronius push service path (e.g., `fronius`).
   - `INFLUX_TOKEN`: Leave this empty for now; you will obtain the token during the InfluxDB setup.

### InfluxDB Setup

1. Run the stack with the following command:

   ```bash
   docker-compose up -d
   ```

2. Once the InfluxDB service is running, navigate to `http://localhost:8086` in your browser.
3. Complete the InfluxDB setup:

   - Set up your organization and bucket as specified in the `.env` file.
   - After setting up, navigate to the **API Tokens** section in InfluxDB and create a token for **Telegraf** with write access to your bucket.

4. Copy the generated token and paste it into the `INFLUX_TOKEN` field in your `.env` file.

### Restart Telegraf Service

After updating the `.env` file with the InfluxDB token, you need to restart the `telegraf` service to apply the changes:

```bash
docker-compose restart telegraf
```

### Fronius Inverter Configuration

To allow your Fronius inverter to send push data to your Telegraf endpoint, follow these steps:

1. Log in to your Fronius inverter's web interface.
2. Navigate to **Settings > Datamanager > Push Services**.
3. Add a new HTTP push service with the following details:

   - **URL**: `http://<your-server-ip>:8094/<TELEGRAF_ENDPOINT_PATH_NAME>` (replace `<your-server-ip>` with your server's IP address and `<TELEGRAF_ENDPOINT_PATH_NAME>` with the value set in your `.env` file, e.g., `fronius`).
   - **Data Format**: JSON
   - **Interval**: Set your desired data transmission interval.

4. Save and activate the push service.

## Running the Stack

To start the services, simply run the following command:

```bash
docker-compose up -d
```

This will start both the InfluxDB and Telegraf services. Telegraf will now listen on the configured HTTP endpoint for push calls from the Fronius inverter.

To stop the stack:

```bash
docker-compose down
```

## Port Configuration

### Local Access

By default, the services run on the following ports:

- InfluxDB: `8086`
- Telegraf: `8092/UDP`, `8094/TCP`, `8125/UDP`

Ensure these ports are not being blocked by your firewall. If you plan to access the services locally only, no further configuration is required.

### Remote Access (Port Forwarding)

If you want to expose your Telegraf endpoint to the internet (e.g., if your Fronius inverter is on a different network), you will need to open or forward ports:

- **Port forwarding**: Forward port `8094` (Telegraf HTTP endpoint) from your router to the machine running Docker.
- **Firewall rules**: Ensure that the ports (e.g., `8094` for Telegraf) are open for external access.

**Note:** Exposing services to the internet can present security risks. Consider using HTTPS and securing your Telegraf endpoint with basic authentication or TLS for secure data transmission.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
