# AI Support Ticket Classifier

A powerful, free machine learning system that automatically classifies support tickets using **GROQ API** and **Llama 3.3** model.

![Python](https://img.shields.io/badge/Python-3.8+-blue.svg)
![License](https://img.shields.io/badge/License-MIT-green.svg)
![GROQ API](https://img.shields.io/badge/GROQ-Free%20API-orange.svg)

## Features

-  **Llama 3.3 Model** - State-of-the-art LLM for classification
-  **Fast Processing** - Quick ticket analysis
-  **EDA Included** - Complete data exploration
-  **Modular Design** - Easy to customize
-  **Jupyter Notebooks** - Well-documented code examples

##  Project Structure

```
AI_Support_Ticket_Classifier/
│
├──  GROQ_INTEGRATION.ipynb        # GROQ API setup & connection
├──  API_FILE_CREATION.ipynb       # Environment configuration
├──  TICKET_CLASSIFIER_FUNCTION.ipynb  # Main classifier logic
├──  EDA.ipynb                     # Exploratory data analysis
│
├── .gitignore                       # Exclude sensitive files
├── .env.example                     # Environment variables template
├── requirements.txt                 # Python dependencies
├── LICENSE                          # MIT License
└── README.md                         # This file
```

##  Quick Start

### Prerequisites
- Python 3.8 or higher
- GitHub account
- GROQ API key (free)

###  Clone the Repository

```bash
git clone https://github.com/yusravekriwala/AI_Support_Ticket_Classifier
cd AI_Support_Ticket_Classifier
```

###  Create Virtual Environment

```bash
# Create virtual environment
python -m venv venv

# Activate it
# On macOS/Linux:
source venv/bin/activate

# On Windows:
venv\Scripts\activate
```

###  Install Dependencies

```bash
pip install -r requirements.txt
```

###  Set Up Environment Variables

```bash
# Copy the example file
cp .env.example .env

# Edit .env and add your GROQ API key
# (Open .env with any text editor)
```

###  Get Free GROQ API Key

1. Visit **[GROQ Console](https://console.groq.com/keys)**
2. Sign up (free)
3. Create new API key
4. Copy the key and paste it in `.env` file:
   ```
   GROQ_API_KEY=your_api_key_here
   ```

###  Run Jupyter Notebooks

```bash
jupyter notebook
```

Then open any `.ipynb` file and run the cells!

##  Notebook Descriptions

### 1. **GROQ_INTEGRATION.ipynb**
Sets up and tests GROQ API connection
- Load API credentials
- List available models
- Test connection

### 2. **API_FILE_CREATION.ipynb**
Creates and manages `.env` file for secure API key storage
- Create `.env` file
- Load environment variables
- Verify API key

### 3. **TICKET_CLASSIFIER_FUNCTION.ipynb**
Main classifier implementation
- Classify support tickets
- Process batch data
- Handle different ticket types

### 4. **EDA.ipynb**
Exploratory data analysis
- Dataset statistics
- Data visualization
- Pattern analysis

##  Technologies Used

| Technology | Purpose |
|-----------|---------|
| **GROQ API** | Free LLM API access |
| **Llama 3.3** | Large language model |
| **Python** | Programming language |
| **Jupyter** | Interactive notebooks |
| **Pandas** | Data manipulation |
| **NumPy** | Numerical computing |

##  Dependencies

```
groq>=0.9.0                  # GROQ API client
python-dotenv>=1.0.0         # Environment variables
pandas>=1.5.0                # Data analysis
jupyter>=1.0.0               # Notebook environment
numpy>=1.20.0                # Numerical computing
scikit-learn>=1.0.0          # Machine learning utilities
```

##  Security

 **IMPORTANT SECURITY NOTES:**

1. **Never commit `.env` file** - It contains your API key
2. **Use `.env.example`** - As a template for your local `.env`
3. **Regenerate keys if exposed** - Visit GROQ console to create new ones
4. **Keep API keys private** - Don't share them in issues or PRs

##  How to Use

### Basic Example

```python
from groq import Groq
from dotenv import load_dotenv
import os

# Load environment variables
load_dotenv()
api_key = os.getenv('GROQ_API_KEY')

# Create client
client = Groq(api_key=api_key)

# Classify a ticket
response = client.chat.completions.create(
    model="llama-3.3-70b-versatile",
    messages=[
        {"role": "user", "content": "Classify this support ticket: Customer can't login"}
    ]
)

print(response.choices[0].message.content)
```

##  Tips & Tricks

- **Rate Limits**: GROQ offers generous free tier limits
- **Model Selection**: Llama 3.3-70b is best for classification
- **Batch Processing**: Process multiple tickets efficiently
- **Error Handling**: Always wrap API calls in try-except blocks

##  Troubleshooting

### "API key not found"
- Check `.env` file exists in project root
- Verify `GROQ_API_KEY=` format (no spaces)
- Restart Jupyter kernel after editing `.env`

### "ModuleNotFoundError"
```bash
# Make sure virtual environment is activated and install dependencies
pip install -r requirements.txt
```

### "Rate limit exceeded"
- Wait a few moments before retrying
- Check GROQ console for usage stats
- Free tier has generous limits, shouldn't hit them often

##  Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

##  License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

##  Author

**Yusra Imran Vekriwala**
- GitHub: [@yusravekriwala](https://github.com/yusravekriwala)
- Email: yusravekriwala@gmail.com

## Acknowledgments

- **GROQ** - For free API access and Llama models
- **Meta** - For Llama 3.3 model
- **Open Source Community** - For amazing tools and libraries

##  Support

Have questions? Create an issue on GitHub or check GROQ documentation at [console.groq.com](https://console.groq.com)

---

 **If you find this helpful, please star the repository!**

**Last Updated**: August 2026
