
## 🛠️ `springdevpro.com`: Enterprise Spring AI & LLM Architecture

**Live Site:** [https://springdevpro.com](https://springdevpro.com)

-----

### 🌟 Project Overview

`SpringDevPro` is the official content repository for the springdevpro.com technical blog. This platform serves as a premier resource for developers and architects focusing on **Spring Framework integration with modern Artificial Intelligence (AI) and Large Language Models (LLMs)**.

Authored by a 20-year veteran enterprise architect, our mission is to move beyond basic tutorials and provide **enterprise-grade, production-ready insights** into building scalable, intelligent applications using the Spring ecosystem.

### ✨ Core Focus & Value Proposition

Our content is specifically tailored for experienced Java developers and architects seeking to master the AI frontier.

  * **Spring AI Deep Dives:** Detailed tutorials on the `Spring AI` project, covering data source integration, output parsing, and model selection.
  * **Architecture for LLMs:** Guidance on designing robust, scalable microservices and cloud-native applications that integrate LLMs (e.g., prompt engineering, securing API keys, observability).
  * **RAG (Retrieval-Augmented Generation):** Practical examples on building effective RAG pipelines using Vector Databases (e.g., Milvus, Chroma) and Spring Boot.
  * **LLMOps & Deployment:** Strategies for managing the lifecycle of AI-powered features, covering deployment on AWS and leveraging cloud services.

### 📚 Content Structure

The repository is organized by content type:

| Directory | Description | Status |
| :--- | :--- | :--- |
| `content/posts/` | All primary technical articles, tutorials, and deep dives. | Active |
| `content/courses/` | Structured materials for paid training courses (e.g., Spring AI Masterclass). | Planning |
| `static/` | Static assets (images, downloadable PDFs, cheatsheets). | Active |

### ⚙️ Technology Stack

This project leverages the efficiency and speed of the Jamstack architecture.

| Component | Technology | Role |
| :--- | :--- | :--- |
| **Framework** | Hugo (Go-based) | High-speed Static Site Generation (SSG) |
| **Theme** | Blowfish (Submodule) | Professional, feature-rich theme for clean UI/UX |
| **Deployment** | Cloudflare Pages | Global CDN hosting and continuous deployment |
| **Source Control**| GitHub | Content management and version control |

### 🚀 Getting Started (Run Locally)

To set up the `SpringDevPro` environment on your local machine:

#### Prerequisites

1.  [Install Hugo](https://gohugo.io/installation/) (Extended version recommended).
2.  Install [Go programming language](https://go.dev/doc/install) (if needed for some Hugo modules).
3.  Install npm/Node.js (for theme development dependencies).

#### Setup Steps

1.  **Clone the Repository:**

    ```bash
    git clone --recurse-submodules git@github.com:YourUsername/springdevpro.git
    cd springdevpro
    ```

    *(Note: The `--recurse-submodules` command is crucial for cloning the Blowfish theme.)*

2.  **Install Dependencies (Theme-related):**

    ```bash
    # This step is required by the Blowfish theme
    npm install
    ```

3.  **Run Hugo Server:**

    ```bash
    hugo server -D
    ```

    The site will be available at `http://localhost:1313`.

### ✍️ Contributing

We welcome feedback and contributions from the community. If you find a typo, outdated code, or want to suggest a new topic:

1.  **Fork** the repository.
2.  Create a new feature branch (`git checkout -b feature/topic-suggestion`).
3.  Make your changes and commit (`git commit -m 'feat: added new RAG tutorial outline'`).
4.  Push to the branch (`git push origin feature/topic-suggestion`).
5.  Open a **Pull Request** explaining your changes.

For security vulnerabilities or major technical contributions, please contact the author directly.

### 📜 License

Distributed under the MIT License. See `LICENSE` for more information.

### 📧 Contact

**Author:** Jeff Taakey (Principal Architect)
**Email:** support@springdevpro.com

-----
