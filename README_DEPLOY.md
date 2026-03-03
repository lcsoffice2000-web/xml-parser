# 🚀 מדריך התקנה ופריסה - המרת XML לאקסל

## סקירה כללית
מערכת web להמרת קבצי חשבוניות XML לפורמט Excel. ניתן לפרוס במספר דרכים.

---

## אפשרות 1: פריסה מקומית (המהיר ביותר להתחלה)

### דרישות מקדימות
- Python 3.8 ומעלה
- pip

### שלבי התקנה

1. **התקן תלויות:**
```bash
cd web-app
pip install -r requirements.txt
```

2. **העתק את הקוד המלא:**
קובץ `app.py` צריך להכיל את כל הלוגיקה מ-`xml_to_excel_parser.py`

3. **הרץ את השרת:**
```bash
python app.py
```

4. **פתח בדפדפן:**
```
http://localhost:5000
```

---

## אפשרות 2: פריסה ל-Render.com (חינם!)

### שלבים:

1. **צור חשבון ב-Render.com**
   - לך ל-https://render.com
   - הרשם בחינם

2. **צור Web Service חדש:**
   - לחץ על "New +" → "Web Service"
   - בחר "Deploy from Git repository" או "Deploy manually"

3. **הגדרות:**
   ```
   Name: xml-to-excel-converter
   Environment: Python 3
   Build Command: pip install -r requirements.txt
   Start Command: gunicorn app:app
   ```

4. **העלה קבצים:**
   - `app.py` (עם הקוד המלא)
   - `index.html`
   - `requirements.txt`

5. **Deploy!**
   - Render יתן לך URL כמו: `https://xml-to-excel-converter.onrender.com`

---

## אפשרות 3: פריסה ל-PythonAnywhere (חינם!)

### שלבים:

1. **צור חשבון ב-PythonAnywhere:**
   - לך ל-https://www.pythonanywhere.com
   - הרשם לחשבון חינמי

2. **העלה קבצים:**
   - Files → Upload
   - העלה את כל הקבצים מתיקיית `web-app`

3. **צור Web App:**
   - Web → Add a new web app
   - בחר Flask
   - Python 3.10

4. **הגדר WSGI:**
   ערוך את `/var/www/yourusername_pythonanywhere_com_wsgi.py`:
   ```python
   import sys
   path = '/home/yourusername/web-app'
   if path not in sys.path:
       sys.path.append(path)
   
   from app import app as application
   ```

5. **Reload Web App**

6. **גש ל-URL שלך:**
   ```
   https://yourusername.pythonanywhere.com
   ```

---

## אפשרות 4: פריסה ל-Railway.app

### שלבים:

1. **צור חשבון ב-Railway:**
   - https://railway.app
   - התחבר עם GitHub

2. **העלה פרויקט:**
   - New Project → Deploy from GitHub repo
   - או: העלה ידנית את התיקייה

3. **Railway יזהה אוטומטית:**
   - Python project
   - יתקין את requirements.txt
   - ירוץ עם gunicorn

4. **קבל URL:**
   - Railway יספק לך domain אוטומטי
   - או קשר domain משלך

---

## מבנה קבצים נדרש

```
web-app/
├── app.py                 # Flask backend (קוד מלא)
├── index.html            # Frontend UI
├── requirements.txt      # Python dependencies
└── README_DEPLOY.md      # המדריך הזה
```

---

## קוד מלא ל-app.py

העתק את הקוד הבא ל-`app.py`:

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
from flask import Flask, request, send_file, render_template_string
from flask_cors import CORS
import xml.etree.ElementTree as ET
import openpyxl
from openpyxl import Workbook
import re
import io

app = Flask(__name__)
CORS(app)

# קרא את ה-HTML
with open('index.html', 'r', encoding='utf-8') as f:
    HTML_TEMPLATE = f.read()

@app.route('/')
def index():
    return render_template_string(HTML_TEMPLATE)

@app.route('/api/convert', methods=['POST'])
def convert():
    try:
        if 'file' not in request.files:
            return 'No file uploaded', 400
        
        file = request.files['file']
        if not file.filename.endswith('.xml'):
            return 'Please upload an XML file', 400
        
        xml_content = file.read().decode('utf-8')
        data_rows = parse_xml_invoice(xml_content)
        excel_file = create_excel_in_memory(data_rows)
        
        output_filename = file.filename.replace('.xml', '.xlsx')
        
        return send_file(
            excel_file,
            mimetype='application/vnd.openxmlformats-officedocument.spreadsheetml.sheet',
            as_attachment=True,
            download_name=output_filename
        )
    except Exception as e:
        return f'Error: {str(e)}', 500

# [הוסף כאן את כל הפונקציות מ-xml_to_excel_parser.py]
# parse_customer_name, parse_description, parse_xml_invoice, create_excel_in_memory

if __name__ == '__main__':
    app.run(debug=True, host='0.0.0.0', port=5000)
```

---

## בדיקה מקומית

```bash
# התקן
pip install -r requirements.txt

# הרץ
python app.py

# בדוק
curl http://localhost:5000
```

---

## טיפים

### 1. אבטחה
- הוסף הגבלת גודל קובץ:
  ```python
  app.config['MAX_CONTENT_LENGTH'] = 16 * 1024 * 1024  # 16MB
  ```

### 2. ביצועים
- השתמש ב-gunicorn עם workers:
  ```bash
  gunicorn -w 4 -b 0.0.0.0:5000 app:app
  ```

### 3. HTTPS
- Render, Railway, PythonAnywhere נותנים HTTPS חינם
- לא צריך להגדיר כלום!

---

## פתרון בעיות

### שגיאה: "Module not found"
```bash
pip install -r requirements.txt --upgrade
```

### שגיאה: "Port already in use"
```bash
# שנה את הפורט
python app.py --port 8000
```

### Excel לא נוצר
- בדוק logs
- ודא ש-openpyxl מותקן
- בדוק שה-XML תקין

---

## תמיכה

יצרת את המערכת הזו? יש בעיה?
- בדוק את ה-logs
- ודא שכל הקבצים הועלו
- נסה להריץ מקומית קודם

---

## רישיון
Smart Business © 2026
