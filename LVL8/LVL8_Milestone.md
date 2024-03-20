# Internationalization (i18n) and Localization (l10n) in React.js

## Objective

The objective of this task is to implement internationalization and localization features in a React.js application to make it accessible and user-friendly for a global audience. This involves enabling dynamic translation of content, localizing date and time formats, and providing the ability to switch between different locales.

## Dynamic Content Translation

To achieve dynamic content translation, follow these steps:

1. **Create Language Files or Dictionaries**: Develop language files or dictionaries that store translations for different languages. Each file should contain key-value pairs where keys represent the original text in the default language and values represent the translated text in the target language.

![alt text](<Screenshot 2024-03-20 211303.png>)

![alt text](<Screenshot 2024-03-20 211324.png>)

`en.json (english translation file)`

```json
{
  "translation": {
    "assignee": "Assignee",
    "cancel": "Cancel",
    "comment": "Comment",
    "create_new_project": "Create New Project",
    "create_new_task": "Create New Task",
    "description": "Description",
    "Done": "Done",
    "due_date": "Due Date",
    "enter_project_name": "Enter Project Name",
    "In progress": "In Progress",
    "members": "Members",
    "newproject": "New Project",
    "newtask": "New Task",
    "Pending": "Pending",
    "profile": "Profile",
    "projects": "Projects",
    "signout": "Sign out",
    "submit": "Submit",
    "title": "Title",
    "update": "Update",
    "enter_title": "Enter Title",
    "enter_description": "Enter Description",
    "task_details": "Task Details"
  }
}
```

`de.json (germen translation file)`

```json
{
  "translation": {
    "assignee": "Abtretungsempfängerin",
    "cancel": "stornieren",
    "comment": "Kommentar",
    "create_new_project": "Neues Projekt erstellen",
    "create_new_task": "Neue Aufgabe erstellen",
    "description": "Beschreibung",
    "Done": "Erledigt",
    "due_date": "Fälligkeitsdatum",
    "enter_project_name": "Geben Sie den Projektnamen ein",
    "In progress": "Im Gange",
    "members": "Mitglieder",
    "newproject": "Neues Projekt",
    "newtask": "Neue Aufgabe",
    "Pending": "Ausstehend",
    "profile": "Profil",
    "projects": "Projekte",
    "signout": "Abmelden",
    "submit": "Einreichen",
    "title": "Titel",
    "update": "Aktualisieren",
    "enter_title": "Geben Sie den Titel ein",
    "enter_description": "Beschreibung eingeben",
    "task_details": "Aufgabendetails"
  }
}
```

2. **Integration with React.js Application**: Integrate the language files or dictionaries into the React.js application. You can use libraries like `react-i18next` or `react-intl` for this purpose. These libraries provide hooks or components to access translated content within components.

`i18n.ts file`

```ts
import i18n from "i18next";
import { initReactI18next } from "react-i18next";
import LanguageDetector from "i18next-browser-languagedetector";
import deJson from "./locale/de.json";
import enJson from "./locale/en.json";
i18n
  .use(initReactI18next)
  .use(LanguageDetector)
  .init({
    resources: {
      de: { ...deJson },
      en: { ...enJson },
    },
    debug: true,
    fallbackLng: "en",
  });
```

3. **Switching Between Languages**: Implement a mechanism to dynamically switch between languages based on user preferences or system settings. This can be achieved by providing a language selector component that allows users to choose their preferred language. Upon selection, the application should update its state and render content in the chosen language.

`changeLanguage.tsx`

```ts
import React from "react";
import { useTranslation } from "react-i18next";
import { Dialog } from "@headlessui/react";

interface LanguageSelectorDialogProps {
  open: boolean;
  onClose: () => void;
}

const LanguageSelectorDialog: React.FC<LanguageSelectorDialogProps> = ({
  open,
  onClose,
}) => {
  const { i18n } = useTranslation();

  const currentLanguage = i18n.language;

  const changeLanguage = (lng: string) => {
    i18n.changeLanguage(lng);
    onClose(); // Close the dialog after language selection
  };

  return (
    <Dialog
      open={open}
      onClose={onClose}
      className="fixed z-10 inset-0 overflow-y-auto"
    >
      <div className="flex items-center justify-center min-h-screen p-4">
        <Dialog.Overlay className="fixed inset-0 bg-black opacity-30" />

        <div className="relative bg-white rounded-lg px-6 py-4">
          <Dialog.Title className="text-lg font-medium mb-4">
            Select Language
          </Dialog.Title>
          <div className="space-y-2">
            <button
              onClick={() => changeLanguage("en")}
              className={`block w-full text-left py-2 px-4 rounded-md hover:bg-gray-100 focus:outline-none ${
                currentLanguage === "en" ? "bg-gray-100" : ""
              }`}
            >
              English
            </button>
            <button
              onClick={() => changeLanguage("de")}
              className={`block w-full text-left py-2 px-4 rounded-md hover:bg-gray-100 focus:outline-none ${
                currentLanguage === "de" ? "bg-gray-100" : ""
              }`}
            >
              Germany
            </button>
            {/* Add more buttons for other languages as needed */}
          </div>
        </div>
      </div>
    </Dialog>
  );
};

export default LanguageSelectorDialog;
```

`In Appbar.tsx`

```jsx
<LanguageSelectorDialog open={dialogOpen} onClose={handleCloseDialog} />
```

4. **Translation of Components and Text Elements**: Apply translation to various components, text elements, and messages throughout the application. Use the provided hooks or components from the chosen i18n library to fetch translated content and replace the default text with the translated text.

```jsx
<button
  type="button"
  id="newProjectBtn"
  onClick={openModal}
  className="rounded-md bg-blue-600 px-4 py-2 text-sm font-medium text-white hover:bg-opacity-95 focus:outline-none focus-visible:ring-2 focus-visible:ring-white focus-visible:ring-opacity-75"
>
  {t("newproject")}
</button>
```

![alt text](<Screenshot 2024-03-20 211536.png>)

![alt text](<Screenshot 2024-03-20 211548.png>)

## Date and Time Localization

To localize date and time formats, follow these steps:

1. **Identify Cultural Preferences**: Understand the variations in date formats (e.g., MM/DD/YYYY or DD/MM/YYYY) and time formats (12-hour vs. 24-hour clock) prevalent in different cultures and regions.

2. **Utilize Localization Libraries**: Utilize libraries or functions that handle date and time localization. In React.js applications, popular libraries like `date-fns`, `moment.js`, or built-in JavaScript `Intl.DateTimeFormat` can be used for this purpose. These libraries provide methods to format dates and times according to the user's locale.

3. **Integration with React Components**: Integrate the chosen localization library into React components where date and time formatting is required. Use the provided functions or components to format timestamps based on the user's locale.

## Example Implementation

Below is an example of how date and time localization can be implemented in a React.js application:

```jsx
// Task.ts
import React from "react";
import { format } from "date-fns";
import { useTranslation } from "react-i18next";

function App() {
  const { t, i18n } = useTranslation();

  const formatDateForPicker = (
    isoDate: string,
    t: (key: string) => string,
    i18n: any
  ) => {
    const dateObj = new Date(isoDate);
    let localeObject;
    switch (i18n.language) {
      case "de":
        localeObject = "de";
        break;
      default:
        localeObject = "en-US";
    }

    const dateFormatter = new Intl.DateTimeFormat(localeObject, {
      day: "numeric",
      month: "long",
      year: "numeric",
    });

    const formattedDate = dateFormatter.format(dateObj);
    return formattedDate;
  };

  return (
    <div className="sm:ml-4 sm:flex sm:w-full sm:justify-between">
      <div>
        <h2 className="text-base font-bold my-1">{task.title}</h2>
        <p className="text-sm text-slate-500">
          {formatDateForPicker(task.dueDate, t, i18n)}
        </p>
        <p className="text-sm text-slate-500">
          {t("description")}: {task.description}
        </p>
        <p className="text-sm text-slate-500">
          {t("assignee")}: {task.assignedUserName ?? "-"}
        </p>
      </div>
      <button
        className="deleteTaskButton cursor-pointer h-4 w-4 rounded-full my-5 mr-5"
        onClick={(event) => {
          event.preventDefault();
          deleteTask(taskDispatch, projectID ?? "", task);
        }}
      >
        Delete
      </button>
    </div>
  );
}

export default App;
```

![alt text](<Screenshot 2024-03-20 211742.png>)

![alt text](<Screenshot 2024-03-20 211751.png>)

# Locale Switching in React.js Application

## Objective

The objective of this task is to implement a user interface (UI) component or a settings panel that allows users to select their preferred language/locale. Additionally, enable the application to respond dynamically to locale changes, updating content and formatting according to the selected locale. It's essential to ensure that the locale switch is persistent across sessions, providing a smooth and personalized experience for users who want to interact with the application in their preferred language.

## Implementation Steps

### 1. Create Language Selector Component

Create a language selector component that allows users to choose their preferred language. This component could be a dropdown menu, radio buttons, or any other UI element that suits your application's design.

### 2. Utilize i18n Library Hooks

Utilize hooks provided by the i18n library (e.g., `useTranslation` from `react-i18next`) to access translation functions and the current locale within your components.

### 3. Handle Locale Change Events

Implement event handlers to handle locale change events triggered by the user selecting a different language in the language selector component. Upon a locale change event, update the application's state with the new locale and trigger a re-render to reflect the changes.

## Example Implementation

Below is an example implementation of locale switching in a React.js application using `react-i18next`:

`changeLanguage.tsx`

```jsx
import React from "react";
import { useTranslation } from "react-i18next";
import { Dialog } from "@headlessui/react";

interface LanguageSelectorDialogProps {
  open: boolean;
  onClose: () => void;
}

const LanguageSelectorDialog: React.FC<LanguageSelectorDialogProps> = ({
  open,
  onClose,
}) => {
  const { i18n } = useTranslation();

  const currentLanguage = i18n.language;

  const changeLanguage = (lng: string) => {
    i18n.changeLanguage(lng);
    onClose(); // Close the dialog after language selection
  };

  return (
    <Dialog
      open={open}
      onClose={onClose}
      className="fixed z-10 inset-0 overflow-y-auto"
    >
      <div className="flex items-center justify-center min-h-screen p-4">
        <Dialog.Overlay className="fixed inset-0 bg-black opacity-30" />

        <div className="relative bg-white rounded-lg px-6 py-4">
          <Dialog.Title className="text-lg font-medium mb-4">
            Select Language
          </Dialog.Title>
          <div className="space-y-2">
            <button
              onClick={() => changeLanguage("en")}
              className={`block w-full text-left py-2 px-4 rounded-md hover:bg-gray-100 focus:outline-none ${
                currentLanguage === "en" ? "bg-gray-100" : ""
              }`}
            >
              English
            </button>
            <button
              onClick={() => changeLanguage("de")}
              className={`block w-full text-left py-2 px-4 rounded-md hover:bg-gray-100 focus:outline-none ${
                currentLanguage === "de" ? "bg-gray-100" : ""
              }`}
            >
              Germany
            </button>
            {/* Add more buttons for other languages as needed */}
          </div>
        </div>
      </div>
    </Dialog>
  );
};

export default LanguageSelectorDialog;
```

`Appbar.tsx`

```jsx
<LanguageSelectorDialog open={dialogOpen} onClose={handleCloseDialog} />
```

![alt text](<Screenshot 2024-03-20 211848.png>)

![alt text](<Screenshot 2024-03-20 211900.png>)
