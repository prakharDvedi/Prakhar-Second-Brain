# eVakeel_MUJhackX

*Part of [[Projects]] → [[Real Projects]]*

**Repo:** [eVakeel_MUJhackX](https://github.com/prakharDvedi/eVakeel_MUJhackX)
**Visibility:** Public
**Language:** JavaScript

---

# eVakeel - AI-Powered Legal Companion
https://e-vakeel-mu-jhack-x.vercel.app/

**eVakeel** is an intelligent legal advisory platform designed to make legal information accessible to every Indian citizen. Built with cutting-edge AI technology, it provides instant legal guidance, document analysis, and compliance tracking in a user-friendly interface.

<img width="1919" height="981" alt="image" src="https://github.com/user-attachments/assets/15c06e99-39b8-44f8-aaa9-38fb7b0273ed" />

## 🌟 Features

- **AI Legal Advisor**: Interactive chat interface powered by GPT-4o-mini for instant legal guidance, context-aware responses based on Indian law, and support for PDF document context.
- **Document Analysis & Parser**: Upload and analyze legal documents (PDF, PNG, JPEG, WebP) with AI for text extraction, simplified summaries, and risk assessment.
- **Legal Health Score**: A 12-point legal compliance checklist for Indian citizens with interactive guidance and progress tracking.
- **Additional Features**: Real-time WebSocket chat, responsive mobile-first design, Markdown formatting support, and secure session management.

## 🛠️ Technology Stack

### Frontend

- **React 19.1.1** - UI library
- **Vite 7.1.7** - Build tool and dev server
- **React Router DOM 7.9.5** - Routing
- **Framer Motion 12.23.24** - Animations
- **Tailwind CSS 4.1.16** - Styling

### Backend

- **Fastify 5.6.0** - Fast web framework
- **OpenAI SDK 4.73.1** - AI integration
- **WebSocket** - Real-time communication
- **PDF-Parse 1.1.1** - PDF text extraction

## 📁 Project Structure

```
eVakeel_MUJhackX/
│
├── Backend/                    # Fastify backend server
│   ├── config.js              # Configuration (OpenAI API key)
│   ├── server.js              # Main server entry point
│   ├── package.json           # Backend dependencies
│   │
│   ├── routes/               # API route handlers
│   ├── services/             # Business logic services
│   ├── plugins/              # Fastify plugins
│   └── utils/                # Utility functions
│
└── frontend/                  # React frontend application
    ├── src/
    │   ├── pages/           # Page components
    │   ├── components/      # Reusable components
    │   ├── services/        # API service layer
    │   ├── utils/           # Utility functions
    │   ├── App.jsx          # Main app component
    │   └── main.jsx         # Entry point
    │
    ├── public/              # Static assets
    ├── package.json         # Frontend dependencies
    └── vite.config.js       # Vite configuration
```

## 🚀 Quick Start

### Prerequisites

- **Node.js** (v16 or higher)
- **npm** (v8 or higher) or **yarn**
- **OpenAI API Key**

### 1. Clone the Repository

```bash
git clone https://github.com/prakharDvedi/eVakeel_MUJhackX
cd eVakeel_MUJhackX
```

### 2. Backend Setup

```bash
cd Backend
npm install
echo "OPENAI_API_KEY=your-api-key-here" > .env
echo "PORT=5050" >> .env
```

### 3. Frontend Setup

```bash
cd ../frontend
npm install
echo "VITE_API_URL=http://localhost:5050" > .env
```

### 4. Run the Application

#### Start Backend Server

```bash
cd Backend
npm start
```

#### Start Frontend Development Server

```bash
cd ../frontend
npm run dev
```

## 🤝 Contributing

1.  Fork the repository
2.  Create a feature branch (`git checkout -b feature/AmazingFeature`)
3.  Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4.  Push to the branch (`git push origin feature/AmazingFeature`)
5.  Open a Pull Request

## 📝 License

This project is part of the MUJhackX hackathon. Please check the license file for details.

## 👥 Authors

- eVakeel Team - 026 ONE SHOT
- MUJhackX 2024

## 🔮 Future Enhancements

- Multi-language support
- User authentication and profile management
- Document history and saved conversations
- Legal case tracking
- Integration with government portals

---

**Note**: This application is designed for informational purposes and does not replace professional legal advice. Always consult with a qualified lawyer for serious legal matters.

**Related:** [[eVakeel2.0]] — later, fuller iteration of this hackathon project.
