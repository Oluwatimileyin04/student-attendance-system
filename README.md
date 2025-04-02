
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>STUDENT ATTENDANCE SYSTEM | Taraba State University</title>
    <style>
        :root {
            --primary: #2c3e50;
            --secondary: #3498db;
            --success: #27ae60;
            --danger: #e74c3c;
            --light: #ecf0f1;
            --dark: #2c3e50;
            --font-main: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }
        body {
            font-family: var(--font-main);
            background-color: #f5f7fa;
            margin: 0;
            padding: 0;
            color: var(--dark);
        }
        .container {
            max-width: 1000px;
            margin: 20px auto;
            background: white;
            padding: 30px;
            border-radius: 10px;
            box-shadow: 0 0 20px rgba(0, 0, 0, 0.1);
        }
        h1 {
            color: var(--primary);
            text-align: center;
            text-transform: uppercase;
            letter-spacing: 1px;
        }
        .form-group {
            margin-bottom: 20px;
        }
        label {
            display: block;
            margin-bottom: 5px;
            font-weight: 600;
            color: var(--primary);
        }
        input, select {
            width: 100%;
            padding: 12px;
            border: 1px solid #ddd;
            border-radius: 4px;
            font-size: 16px;
        }
        button {
            background-color: var(--secondary);
            color: white;
            border: none;
            padding: 14px 20px;
            border-radius: 4px;
            cursor: pointer;
            font-size: 16px;
            font-weight: 600;
            width: 100%;
            transition: all 0.3s;
        }
        button:hover {
            background-color: #2980b9;
        }
        .status-message {
            padding: 12px;
            border-radius: 4px;
            margin-top: 20px;
            text-align: center;
            font-weight: 600;
        }
        .success { background-color: rgba(39, 174, 96, 0.2); color: var(--success); }
        .error { background-color: rgba(231, 76, 60, 0.2); color: var(--danger); }
    </style>
</head>
<body>
    <div class="container">
        <h1>STUDENT ATTENDANCE SYSTEM</h1>
        
        <!-- Student Login -->
        <div id="studentLogin">
            <div class="form-group">
                <label for="studentMatric">Matric Number</label>
                <input type="text" id="studentMatric" placeholder="TSU/FED/CS/23/1001">
            </div>
            <button onclick="studentLogin()">Login</button>
        </div>
        
        <!-- Attendance Section -->
        <div id="studentAttendance" style="display: none;">
            <h2>Mark Your Attendance</h2>
            <div class="form-group">
                <label for="attendanceCourse">Course</label>
                <select id="attendanceCourse">
                    <option value="">Select Course</option>
                    <option value="MAT201">MAT201 - Mathematics</option>
                    <option value="SEN201">SEN201 - Software Engineering</option>
                </select>
            </div>
            <button onclick="verifyCaptcha()">Verify & Mark Attendance</button>
            <p id="captchaChallenge"></p>
            <input type="text" id="captchaInput" placeholder="Enter answer">
            <button onclick="markAttendance()">Submit Attendance</button>
            <div id="attendanceMessage" class="status-message" style="display: none;"></div>
        </div>
    </div>

    <script>
        let deviceFingerprint = generateFingerprint();
        let attendanceRecords = JSON.parse(localStorage.getItem('attendanceRecords')) || {};

        function generateFingerprint() {
            return navigator.userAgent + navigator.language + screen.width + screen.height;
        }

        function studentLogin() {
            const matric = document.getElementById('studentMatric').value.trim();
            if (!matric.startsWith("TSU/FED/CS/23/")) {
                alert("Invalid matric number format!");
                return;
            }
            sessionStorage.setItem('currentStudent', matric);
            document.getElementById('studentLogin').style.display = "none";
            document.getElementById('studentAttendance').style.display = "block";
        }

        function verifyCaptcha() {
            let num1 = Math.floor(Math.random() * 10);
            let num2 = Math.floor(Math.random() * 10);
            sessionStorage.setItem('captchaAnswer', num1 + num2);
            document.getElementById('captchaChallenge').textContent = `Solve: ${num1} + ${num2} = ?`;
        }

        function markAttendance() {
            let matric = sessionStorage.getItem('currentStudent');
            let course = document.getElementById('attendanceCourse').value;
            let captchaInput = document.getElementById('captchaInput').value;
            let correctCaptcha = sessionStorage.getItem('captchaAnswer');
            let messageElement = document.getElementById('attendanceMessage');

            if (!course) {
                messageElement.textContent = "Please select a course.";
                messageElement.className = "status-message error";
                messageElement.style.display = "block";
                return;
            }

            if (captchaInput != correctCaptcha) {
                messageElement.textContent = "Incorrect CAPTCHA. Try again.";
                messageElement.className = "status-message error";
                messageElement.style.display = "block";
                return;
            }

            if (!attendanceRecords[course]) {
                attendanceRecords[course] = {};
            }

            if (attendanceRecords[course][matric]) {
                messageElement.textContent = "Attendance already marked!";
                messageElement.className = "status-message error";
                messageElement.style.display = "block";
                return;
            }

            if (Object.values(attendanceRecords[course]).some(record => record.device === deviceFingerprint)) {
                messageElement.textContent = "Proxy detected! One device, one attendance.";
                messageElement.className = "status-message error";
                messageElement.style.display = "block";
                return;
            }

            navigator.geolocation.getCurrentPosition(position => {
                let latitude = position.coords.latitude;
                let longitude = position.coords.longitude;

                if (latitude < 7.9 || latitude > 8.1 || longitude < 11.3 || longitude > 11.5) {
                    messageElement.textContent = "Attendance must be marked inside campus.";
                    messageElement.className = "status-message error";
                    messageElement.style.display = "block";
                    return;
                }

                attendanceRecords[course][matric] = {
                    device: deviceFingerprint,
                    timestamp: new Date().toISOString()
                };

                localStorage.setItem('attendanceRecords', JSON.stringify(attendanceRecords));

                messageElement.textContent = "Attendance marked successfully!";
                messageElement.className = "status-message success";
                messageElement.style.display = "block";

                setTimeout(() => location.reload(), 2000);
            }, () => {
                messageElement.textContent = "Location access required!";
                messageElement.className = "status-message error";
                messageElement.style.display = "block";
            });
        }
    </script>
</body>
</html>
