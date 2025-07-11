<img src="/public/logo.png" width="100">

# [Better Form](https://better-form.vercel.app)

A modern, minimalist & open-source alternative to Google Forms. Create beautiful, interactive forms that feel more like conversations. Designed for higher completion rates and better data quality.

## About
- One question at a time conversational UI
- Customizable question types: Text, Multiple Choice, Email, Rating
- Responsive and mobile-friendly design
- CSV export
- Shareable public form links
- Built with Vue 3, Firebase, and Tailwind CSS

## Setup
1. **Clone the repository:**
   ```bash
   git clone https://github.com/leecheeyong/better-form.git
   cd better-form
   ```
2. **Install dependencies:**
   ```bash
   npm install
   # or
   yarn install
   ```
3. **Configure Firebase:**
   - Go to [Firebase Console](https://console.firebase.google.com/) and create a new project.
   - Enable **Authentication** (Email/Password).
   - Set up **Cloud Firestore** in test mode (or with secured rules). 
   - Edit the `src/firebase.js` file, replace with your Firebase credentials

## Todo
- [ ] Connection between Google Sheets & Better Form (Responses Update)
- [ ] Clean up the code
- [ ] Build plugins like webhook, push notifications
- [ ] Turn this into an PWA perhaps

## Contributing
Contributions are welcome! Please [open issues](https://github.com/leecheeyong/better-form/issues) or [pull requests](https://github.com/leecheeyong/better-form/pulls) for improvements and bug fixes.

## License
This project is available as an open source under the terms of the [MIT License](/LICENSE)

