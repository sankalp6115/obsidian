> “Now coming to the technical implementation, we will divide our system into several layers, with each technology serving a specific purpose.
> 
> For the **frontend**, we will use **React** for the web dashboard and **React Native** for the mobile application. We will use **D3.js** for visualizing cyclone and weather data, and **Leaflet.js** for displaying the cyclone's current position, predicted trajectory and affected regions on an interactive map. **Tailwind CSS** will help us build a responsive interface.
> 
> For the **backend**, we will use **FastAPI and Python** because our AI and data-processing pipeline will also be Python-based, making model integration easier.
> 
> We will use **gRPC for communication between our backend services**. The reason for choosing gRPC instead of REST is that our architecture may have multiple services—for example, data ingestion, preprocessing, CNN inference and LSTM prediction—and these services need to communicate frequently and efficiently. gRPC uses **Protocol Buffers**, which provide compact binary serialization and strongly typed service contracts, making communication faster and more structured than typical JSON-based REST communication.
> 
> For the **database**, we will use **PostgreSQL** for structured information such as cyclone metadata and application data, while **InfluxDB** will be used for time-series data such as continuously changing weather and cyclone observations.
> 
> For **AI/ML**, we will use **TensorFlow and PyTorch** for developing our deep-learning models and **scikit-learn** for preprocessing and evaluation. The CNN will process satellite imagery to detect and classify cyclonic systems, while the temporal model will use historical and environmental data to predict the cyclone's future track and intensity.
> 
> For the **datasets**, we will use INSAT-3D and INSAT-3DR satellite imagery from MOSDAC, IMD best-track data, the Digital Typhoon dataset, NOAA HURSAT and OpenWeatherMap meteorological data.
> 
> Finally, for **deployment and monitoring**, we will use **Docker** to containerize our services, **Nginx** as a reverse proxy, **Grafana** for monitoring, **GitHub** for version control, and **Google Colab GPU** resources for training our models.”


> **“Our system continuously receives time-stamped cyclone and meteorological observations. We will use InfluxDB to efficiently store and query this live time-series data, which can then be used for recent-history analysis, LSTM inference, trajectory visualization and monitoring prediction performance.”**
### The gRPC point to remember

If the judge asks **“Why gRPC?”**, don't give a long answer. Say:

> **“We will use gRPC primarily for internal service-to-service communication. REST is simpler and very suitable for public APIs, but gRPC gives us strongly typed contracts, efficient binary serialization and good performance for frequent communication between our ML and backend services. WebSockets are more appropriate for persistent real-time client-server communication, whereas our gRPC use case is backend-to-backend communication.”**

And importantly, **don't say gRPC is simply “faster than REST” as your entire justification**. REST and gRPC solve somewhat different architectural needs. Your strongest argument is **service-to-service communication + typed contracts + efficient serialization**.

For this slide, I’d present it as a **problem → challenge → solution strategy** story rather than reading each box individually.

### Feasibility & Viability — Speech

> “Coming to the feasibility and viability of our solution, we have looked at both whether the system can realistically be built and what challenges we may face during real-world deployment.
> 
> **First, feasibility.** The required data is available from sources such as satellite imagery, meteorological and ocean data, along with geographic information. We will use established AI techniques such as CNNs for image-based cyclone detection and feature extraction, and LSTMs for track and intensity forecasting. The models can also be trained using available cloud and GPU infrastructure, and the final system can be deployed as a web and mobile platform.
> 
> **However, there are some important challenges.** The first is prediction uncertainty because cyclone tracks and intensity can change significantly over time. We also have to deal with false alarms, because both false positives and false negatives can reduce trust in the system. Another challenge is data quality — satellite observations may have missing or low-resolution data because of cloud cover, sensor limitations or delays. Finally, since satellite data can be large and continuously arriving, we need sufficiently low-latency processing for timely alerts.
> 
> **To address these challenges, our viability strategy is to combine multiple sources of information instead of depending on a single source. We will also use probabilistic modelling so that the system can communicate the confidence or uncertainty associated with its predictions.**
> 
> We will use robust validation and threshold tuning to reduce false alarms, and data augmentation and redundancy to make the system more resilient to missing or noisy data.
> 
> Most importantly, we see our AI as **decision support, not a replacement for meteorologists**. The system will act as a high-speed analytical assistant, providing rapid and location-specific insights that can help experts make better-informed decisions.”

### The key logic behind the slide

If the judges start questioning the slide, keep this chain in mind:

**Feasibility**

> “Can we build it?”  
> → Data exists + proven ML techniques + available computing + deployable architecture.

**Challenges**

> “What can go wrong?”  
> → Uncertainty + false alarms + bad/missing data + real-time processing.

**Viability strategy**

> “How will we make it reliable enough to be useful?”  
> → Multi-source data + probabilistic predictions + validation + redundancy + human oversight.

The **most important sentence on this slide** is probably:

> **“We are not claiming that AI will replace the existing forecasting infrastructure; we are proposing an AI-assisted decision-support layer that can process multiple data sources and provide rapid, consistent and location-specific insights.”**

That positioning will make your proposal much more credible when judges ask about existing systems such as IMD.