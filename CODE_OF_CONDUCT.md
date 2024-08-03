pip install flask
project/
├── app.py
└── templates/
    └── index.html
    from flask import Flask, render_template, request, jsonify
import random
import string
import smtplib
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart

app = Flask(__name__)

# إعدادات البريد الإلكتروني
EMAIL_ADDRESS = 'your_email@example.com'
EMAIL_PASSWORD = 'your_password'
SMTP_SERVER = 'smtp.example.com'
SMTP_PORT = 587

def generate_random_code():
    letters = string.ascii_uppercase
    return ''.join(random.choice(letters) for _ in range(6))

def send_email(recipient, code):
    msg = MIMEMultipart()
    msg['From'] = EMAIL_ADDRESS
    msg['To'] = recipient
    msg['Subject'] = 'رمز التحقق الخاص بك'
    
    body = f'رمز التحقق الخاص بك هو: {code}'
    msg.attach(MIMEText(body, 'plain'))
    
    with smtplib.SMTP(SMTP_SERVER, SMTP_PORT) as server:
        server.starttls()
        server.login(EMAIL_ADDRESS, EMAIL_PASSWORD)
        text = msg.as_string()
        server.sendmail(EMAIL_ADDRESS, recipient, text)

@app.route('/')
def index():
    return render_template('index.html')

@app.route('/send_code', methods=['POST'])
def send_code():
    email = request.form['email']
    if not email:
        return jsonify({'error': 'يرجى إدخال البريد الإلكتروني'}), 400

    code = generate_random_code()
    try:
        send_email(email, code)
    except Exception as e:
        return jsonify({'error': str(e)}), 500

    return jsonify({'code': code})

if __name__ == '__main__':
    app.run(debug=True)


<!DOCTYPE html>
<html lang="ar">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>تأكيد البريد الإلكتروني</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            text-align: center;
            padding: 20px;
        }
        .hidden {
            display: none;
        }
        .message {
            padding: 20px;
            margin-top: 20px;
            font-size: 18px;
            font-weight: bold;
        }
        .blue-window {
            background-color: blue;
            color: white;
        }
        .red-window {
            background-color: red;
            color: white;
        }
    </style>
</head>
<body>

    <h1>تأكيد البريد الإلكتروني</h1>
    <input type="email" id="email" placeholder="أدخل البريد الإلكتروني">
    <br><br>
    <button onclick="sendEmail()">تأكيد</button>
    
    <div id="input-code-section" class="hidden">
        <br>
        <input type="text" id="verification-code" placeholder="أدخل الأحرف">
        <br><br>
        <button onclick="verifyCode()">تحقق</button>
    </div>

    <div id="sms-message" class="message hidden"></div>
    <div id="message" class="message hidden"></div>

    <script>
        let generatedCode = '';

        function sendEmail() {
            const email = document.getElementById('email').value;
            if (email === '') {
                alert('يرجى إدخال البريد الإلكتروني');
                return;
            }

            fetch('/send_code', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/x-www-form-urlencoded'
                },
                body: new URLSearchParams({
                    email: email
                })
            })
            .then(response => response.json())
            .then(data => {
                if (data.error) {
                    alert(data.error);
                } else {
                    generatedCode = data.code;
                    const smsMessageDiv = document.getElementById('sms-message');
                    smsMessageDiv.textContent = `تم إرسال الرمز: ${generatedCode} إلى البريد الإلكتروني: ${email}`;
                    smsMessageDiv.classList.remove('hidden');
                    document.getElementById('input-code-section').classList.remove('hidden');
                }
            })
            .catch(error => console.error('Error:', error));
        }

        function verifyCode() {
            const inputCode = document.getElementById('verification-code').value;
            const messageDiv = document.getElementById('message');
            if (inputCode === generatedCode) {
                messageDiv.textContent = 'تم التحقق بنجاح!';
                messageDiv.className = 'message blue-window';
            } else {
                messageDiv.textContent = 'الرمز غير صحيح، يرجى المحاولة مرة أخرى.';
                messageDiv.className = 'message red-window';
            }
            messageDiv.classList.remove('hidden');
        }
    </script>

</body>
</html>


python app.py

    
