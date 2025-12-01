# Temi Quiz Web Project

This project contains a complete solution for running a multiple‑choice and rating‑based quiz on a Temi robot. It includes:

- A **Google Sheet** that stores questions and responses.
- A **Google Apps Script** that serves as a simple REST API for fetching questions and saving answers.
- A **browser‑based admin panel** for adding, editing, enabling/disabling, and deleting quiz questions, and for playing the associated audio files.
- A **user page** designed for the Temi robot which displays one question at a time, plays the associated audio without showing any controls, and sends responses back to the sheet.
- Instructions for hosting the static site on a free platform such as GitHub Pages or Firebase Hosting and configuring Temi to display it.

## 1. Prepare your Google Sheet

1. Create a new Google Sheet and rename the first two tabs as follows:
   - **questions** – holds all quiz questions.
   - **responses** – stores user responses.
2. In `questions` create columns in this order:

   | Column | Purpose                      |
   |------|-----------------------------|
   | A    | `id` – unique numeric identifier (generated when adding questions) |
   | B    | `question` – the text of the question |
   | C    | `type` – `mcq` for multiple choice or `rating` for 1‑10 rating |
   | D    | `option1` – first option (if type = mcq) |
   | E    | `option2` – second option |
   | F    | `option3` – third option |
   | G    | `option4` – fourth option |
   | H    | `audioURL` – URL of the MP3 file for the audio prompt |
   | I    | `enabled` – `TRUE`/`FALSE` to show or hide the question |

3. In `responses` create columns:

   | Column | Purpose                 |
   |------|------------------------|
   | A    | `timestamp` – date and time of submission |
   | B    | `questionId` – ID of the question answered |
   | C    | `answer` – selected option text or rating number |

## 2. Create the Google Apps Script API

1. In your Sheet click **Extensions → Apps Script**. Delete the default code and replace it with the following script:

```javascript
function doGet() {
  const sheet = SpreadsheetApp.getActive().getSheetByName('questions');
  const data = sheet.getDataRange().getValues();
  const questions = [];
  for (let i = 1; i < data.length; i++) {
    questions.push({
      id: data[i][0],
      question: data[i][1],
      type: data[i][2],
      option1: data[i][3],
      option2: data[i][4],
      option3: data[i][5],
      option4: data[i][6],
      audioURL: data[i][7],
      enabled: data[i][8],
    });
  }
  return ContentService.createTextOutput(JSON.stringify(questions))
    .setMimeType(ContentService.MimeType.JSON);
}

function doPost(e) {
  const action = e.parameter.action;
  if (action === 'addQuestion') return addQuestion(e);
  if (action === 'editQuestion') return editQuestion(e);
  if (action === 'deleteQuestion') return deleteQuestion(e);
  if (action === 'saveResponse') return saveResponse(e);
  return ContentService.createTextOutput('Invalid action');
}

function addQuestion(e) {
  const sheet = SpreadsheetApp.getActive().getSheetByName('questions');
  const id = new Date().getTime();
  sheet.appendRow([
    id,
    e.parameter.question,
    e.parameter.type,
    e.parameter.option1,
    e.parameter.option2,
    e.parameter.option3,
    e.parameter.option4,
    e.parameter.audioURL,
    e.parameter.enabled
  ]);
  return ContentService.createTextOutput('Added');
}

function editQuestion(e) {
  const id = Number(e.parameter.id);
  const sheet = SpreadsheetApp.getActive().getSheetByName('questions');
  const data = sheet.getDataRange().getValues();
  for (let i = 1; i < data.length; i++) {
    if (data[i][0] === id) {
      sheet.getRange(i + 1, 2).setValue(e.parameter.question);
      sheet.getRange(i + 1, 3).setValue(e.parameter.type);
      sheet.getRange(i + 1, 4).setValue(e.parameter.option1);
      sheet.getRange(i + 1, 5).setValue(e.parameter.option2);
      sheet.getRange(i + 1, 6).setValue(e.parameter.option3);
      sheet.getRange(i + 1, 7).setValue(e.parameter.option4);
      sheet.getRange(i + 1, 8).setValue(e.parameter.audioURL);
      sheet.getRange(i + 1, 9).setValue(e.parameter.enabled);
      break;
    }
  }
  return ContentService.createTextOutput('Updated');
}

function deleteQuestion(e) {
  const id = Number(e.parameter.id);
  const sheet = SpreadsheetApp.getActive().getSheetByName('questions');
  const data = sheet.getDataRange().getValues();
  for (let i = 1; i < data.length; i++) {
    if (data[i][0] === id) {
      sheet.deleteRow(i + 1);
      break;
    }
  }
  return ContentService.createTextOutput('Deleted');
}

function saveResponse(e) {
  const sheet = SpreadsheetApp.getActive().getSheetByName('responses');
  sheet.appendRow([
    new Date(),
    e.parameter.questionId,
    e.parameter.answer
  ]);
  return ContentService.createTextOutput('Saved');
}
```

2. **Deploy the script as a web app**. At the top‑right, click **Deploy → New deployment**. When asked for the type, choose **Web app**, provide a short description and set:

   - **Execute as:** **Me** (your Google account). Running as the script owner ensures it has access to your Sheet.
   - **Who has access:** **Anyone** (or **Anyone with Google Account** for more security). This allows your static site to call the API without requiring users to log into Google.

   Finally click **Deploy** and copy the long URL ending in `/exec` — this is your API endpoint【601492138147531†L426-L436】【329517136262414†L57-L66】.

3. Whenever you modify the script, redeploy the web app to create a new version. The web‑app URL remains the same.

## 3. Update the API URL in the web pages

In both `admin.html` and `user.html` files, locate the line:

```javascript
const API_URL = 'REPLACE_WITH_YOUR_WEBAPP_URL';
```

Replace the placeholder with your copied web‑app URL. Save the files.

## 4. Host the web pages for free

There are two popular options:

### Option A – GitHub Pages

1. Create a new GitHub repository (public or private). Copy `admin.html`, `user.html` and this `README.md` into the repository.
2. Commit and push the files. In the repository settings, enable **GitHub Pages** for the `main` or `docs` branch. GitHub Pages provides a free static site with up to 1 GB of storage and a soft bandwidth limit of 100 GB per month【982512377197451†L91-L103】.
3. Your site will be available at `https://username.github.io/repository-name/`. The admin page will be at `.../admin.html` and the user/Temi page at `.../user.html`.

### Option B – Firebase Hosting

1. Go to the [Firebase console](https://console.firebase.google.com/) and create a new project. Register a web app in your project.
2. Install the Firebase CLI on your computer (`npm install -g firebase-tools`) and run `firebase login` to authenticate.
3. Initialise hosting in the project directory: `firebase init` and choose **Hosting: Configure and deploy**. When asked for a directory to use for public files, enter `.` (current folder). When asked to configure as a single‑page app, choose **No**.
4. Place `admin.html`, `user.html` and any audio files into this directory. Run `firebase deploy` to upload. The Spark free tier includes 10 GB of storage and up to 360 MB/day of data transfer【519529466448840†L189-L193】.
5. Your site will be accessible at a URL like `https://PROJECT-ID.web.app/`.

### Hosting audio files

If you have MP3 files for each question, place them in an `audio/` directory within your GitHub or Firebase project and commit them. The URL for a file `audio/question1.mp3` will be `https://.../audio/question1.mp3`. Paste this URL into the **Audio URL** field when adding or editing a question. Keeping audio files in the same project simplifies hosting.

## 5. Using the Admin Panel

1. Open `admin.html` in your hosted site or by double‑clicking it locally (the pages can be tested locally as long as the API URL is correct).
2. The table automatically loads all questions from the Google Sheet and displays them in a clean format. You can:
   - **Play** the audio using the **Play** button.
   - **Toggle Enabled**: Hide or show a question on the Temi page using the checkbox.
   - **Edit**: Click **Edit**, update the fields, then submit to save.
   - **Delete**: Remove a question completely.
3. To add a question:
   - Enter the question text.
   - Select **Multiple Choice** or **Rating** from the **Type** drop‑down. If you choose **Multiple Choice**, four option fields appear; fill them all.
   - Provide the Audio URL (e.g., `https://your-site/audio/myquestion.mp3`).
   - Tick **Enabled** if you want the question to appear on Temi.
   - Click **Add Question**. The script assigns a unique ID and writes the row to the `questions` sheet.

## 6. Using the User (Temi) Page

1. Open `user.html` from your hosted site. The page fetches only the enabled questions and shows them one by one.
2. When a question appears, its audio plays automatically (there is no visible audio player). If it is a multiple‑choice question, four buttons appear; if it is a rating question, ten buttons (1–10) appear.
3. After the user selects an option or rating, the response is sent to the `responses` sheet and the next question appears. After the last question, a completion message is shown.

## 7. Configure Temi to display the quiz

1. Log in to [Temi Center](https://center.robotemi.com/) with your Temi account. Scan the QR code from the mobile app to register your robot if you have not already done so.
2. In the home‑screen customization settings, choose **Web page** and provide the URL of your `user.html` page. According to the Temi Center user guide, selecting “Web page” will display that page on Temi’s screen【250560199801497†L134-L147】.
3. Save the configuration. The Temi robot will roam around your event space and display your quiz page on its screen, automatically playing each question’s audio as participants answer.

## 8. Maintaining and extending the quiz

- **Updating questions:** Use the Admin panel to add, edit, enable/disable or delete questions at any time. The user page always fetches the latest list when loaded.
- **Analytics:** Because responses are stored in the `responses` sheet, you can use Google Sheets charts or Apps Script to analyse data in real time.
- **Security:** For casual events you can set “Anyone” access. For more controlled environments, choose “Anyone with Google Account” for the web app and restrict hosting via basic authentication or a password on the admin page.

## Citations

- The Google Apps Script “Web Apps” guide explains how to deploy a script as a web app: click **Deploy > New deployment**, select **Web app**, configure settings, and deploy【601492138147531†L426-L436】.  Another setup guide emphasises selecting **Execute as: Me** and **Who has access: Anyone** during deployment【329517136262414†L57-L66】.
- You can display a web page on Temi’s home screen through Temi Center by selecting the **Web page** option and providing the URL【250560199801497†L134-L147】.
- GitHub Pages offers free static hosting with a 1 GB size limit and 100 GB/month bandwidth cap【982512377197451†L91-L103】.
- Firebase Hosting’s free Spark tier includes up to 10 GB of storage and 360 MB/day of data transfer【519529466448840†L189-L193】.
