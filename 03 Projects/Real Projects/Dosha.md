# Dosha

*Part of [[Projects]] → [[Real Projects]]*

**Repo:** [Dosha](https://github.com/prakharDvedi/Dosha)
**Visibility:** Public
**Language:** JavaScript

---

# 🧘‍♀️ Dosha Assessment Web Application

A comprehensive Ayurvedic dosha assessment tool built with React and Tailwind CSS. This interactive web application helps users discover their unique mind-body constitution through a series of carefully crafted questions.

## 🌟 Features

- **Interactive Slider Interface** - Beautiful, responsive sliders with light aesthetic colors
- **45 Comprehensive Questions** - 15 questions each for Vata, Pitta, and Kapha doshas
- **Real-time Progress Tracking** - Visual progress bars and question counters
- **Dynamic Scoring System** - Live calculation of dosha percentages
- **Detailed Results View** - Personalized recommendations and constitution analysis
- **Educational Content** - Learn about each dosha with characteristics and balancing tips
- **Responsive Design** - Optimized for desktop, tablet, and mobile devices
- **Modern UI/UX** - Clean, intuitive interface with smooth animations

## 🚀 Live Demo

Visit the live application: [Dosha Assessment](https://github.com/prakharDvedi/Dosha)

## 🛠️ Technologies Used

- **React 18** - Modern React with hooks and functional components
- **Vite** - Fast build tool and development server
- **Tailwind CSS 3.4.17** - Utility-first CSS framework
- **JavaScript ES6+** - Modern JavaScript features
- **PostCSS** - CSS processing and optimization

## 📦 Installation & Setup

1. **Clone the repository**
   ```bash
   git clone https://github.com/prakharDvedi/Dosha.git
   cd Dosha
   ```

2. **Install dependencies**
   ```bash
   npm install
   ```

3. **Start development server**
   ```bash
   npm run dev
   ```

4. **Build for production**
   ```bash
   npm run build
   ```

5. **Preview production build**
   ```bash
   npm run preview
   ```

## 📁 Project Structure

```
dosha-assessment/
├── src/
│   ├── components/
│   │   ├── SliderQuestion.jsx    # Interactive question slider component
│   │   ├── DoshaCard.jsx         # Individual dosha display card
│   │   └── ResultView.jsx        # Results and recommendations view
│   ├── pages/
│   │   └── DoshaAssessment.jsx   # Main application page
│   ├── utils/
│   │   └── doshaData.js          # Question database and dosha information
│   ├── App.jsx                   # Root application component
│   ├── main.jsx                  # Application entry point
│   └── index.css                 # Global styles and Tailwind imports
├── public/
├── dist/                         # Production build output
├── package.json
├── tailwind.config.js
├── postcss.config.js
└── vite.config.js
```

## 🎯 How It Works

1. **Welcome Screen** - Introduction to Ayurveda and dosha assessment
2. **Question Flow** - 45 randomized questions with slider inputs (Never to Always)
3. **Progress Tracking** - Real-time progress indication and question navigation
4. **Score Calculation** - Dynamic scoring based on user responses
5. **Results Display** - Comprehensive analysis with:
   - Dosha percentages and dominant constitution
   - Detailed characteristics for each dosha
   - Personalized balancing recommendations
   - Educational content about Ayurveda

## 🔧 Key Components

### SliderQuestion Component
- Interactive range slider (0-4 scale)
- Visual feedback with emojis and colors
- Progress tracking and question navigation
- Responsive design with touch support

### DoshaCard Component
- Individual dosha information display
- Progress bars showing relative percentages
- Highlighting for dominant dosha
- Educational content and characteristics

### ResultView Component
- Comprehensive results analysis
- Constitution type determination (Pure, Dual, Tri-Doshic)
- Personalized recommendations
- Balancing tips and lifestyle suggestions

## 🎨 Design Features

- **Light Aesthetic Colors** - Soft gradients and pastel color palette
- **Smooth Animations** - CSS transitions and hover effects
- **Modern Typography** - Clean, readable font choices
- **Intuitive Navigation** - Clear progress indicators and controls
- **Mobile-First Design** - Responsive layout for all screen sizes

## 📊 Dosha Information

### Vata (Air & Space)
- Characteristics: Creative, energetic, quick-thinking
- Imbalances: Anxiety, restlessness, digestive issues
- Balancing: Routine, warm foods, grounding practices

### Pitta (Fire & Water)
- Characteristics: Focused, ambitious, strong digestion
- Imbalances: Anger, inflammation, overheating
- Balancing: Cooling foods, moderation, stress management

### Kapha (Earth & Water)
- Characteristics: Stable, nurturing, strong immunity
- Imbalances: Sluggishness, weight gain, congestion
- Balancing: Activity, light foods, stimulation

## 🚀 Deployment

The application is optimized for deployment on:
- **Vercel** - Automatic deployments from GitHub
- **Netlify** - Continuous deployment with form handling
- **GitHub Pages** - Static site hosting
- **Any static hosting service**

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- Ayurvedic principles and traditional knowledge
- Modern web development best practices
- React and Tailwind CSS communities
- Open source contributors

## 📞 Contact

**Prakhar Dwivedi**
- GitHub: [@prakharDvedi](https://github.com/prakharDvedi)
- Project Link: [https://github.com/prakharDvedi/Dosha](https://github.com/prakharDvedi/Dosha)

---

*This assessment is for educational purposes only and should not replace professional medical advice.*
