# Archive
<!DOCTYPE html>
<html lang='fa'>
<head>
<meta charset='UTF-8'>
<meta name='viewport' content='width=device-width, initial-scale=1.0'>
<title>Nian Electronic</title>
<style type='text/css'>
body {font: 100%/1.4 Tahoma, Geneva, sans-serif;background: #273965;margin: 0;padding: 0;color: #FFF;}
.container {width: 90%;max-width: 800px;min-width: 300px;background: #FFF;margin: 0 auto;padding: 20px;box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);border-radius: 12px;}
.header {width: 100%;background: #4f6d9f;text-align: center;color: #FFF;border-radius: 12px 12px 0 0;margin-bottom: 20px;}
.header h1 {margin: 0;padding: 20px;}
.content {font-size: 15px;text-align: left; color: #000;}
input[type='button'], input[type='checkbox'], select {font-weight: bold;padding: 10px 20px;border-radius: 12px;border: 1px solid #4CAF50;cursor: pointer;margin: 5px;background-color: #4CAF50; color: #FFF;}
input[type='button']:hover {background-color: #45a049;}
input[type='button'].toggled {background-color: #0000FF;}
select {background-color: #FFF;border: 1px solid #4CAF50;color: #000;}
.footer {padding: 10px 0;font-size: 12px;background: #4f6d9f;color: #FFF;text-align: center;border-radius: 0 0 12px 12px;margin-top: 20px;}
.tables-container {display: flex;justify-content: center;gap: 20px;flex-wrap: wrap;}
#controlPanelSystem {width: 100%;background-color: #4CAF50;color: #FFF;font-weight: bold;text-align: center;padding: 10px;border-radius: 12px;}
.status {font-size: 18px;font-weight: bold;margin-top: 10px; color: #FFF;}
.status-off {background-color: #FF0000;}
.status-on {background-color: #4CAF50;}
#progress-bar {width: 100%; background-color: #f3f3f3; border-radius: 12px; overflow: hidden; cursor: pointer; box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2); height: 40px;}
#progress-bar-fill {height: 100%; width: 0; background-color: #4CAF50; text-align: center; line-height: 40px; color: white; transition: width 0.3s ease; font-weight: bold; font-size: 1.2em;}
#progress-bar.locked {pointer-events: none; opacity: 0.6;}
</style>
</head>
<body>
<div class='container'>
<div class='header'>
<h1>Nian Electronic</h1>
</div>
<div class='content'>
<div class='tables-container'>
<table width='60%' border='3' align='center' bordercolor='#4f6d9f' cellspacing='10' style='text-align:center;border-radius:12px;'>
<tr style='color:black;font-weight:bold;font-size:18px;'>
<td><select id='temperatureSelect' onchange='handleTemperatureChange()'></select></td>
<td><input type='checkbox' id='autoMode' name='autoMode' onchange='handleCheckboxChange()'> اتوماتیک</td>
</tr>
</table>
<div id='progress-bar'>
<div id='progress-bar-fill'>0%</div>
</div>
</div>
<table width='100%' border='3' align='center' bordercolor='#4f6d9f' cellspacing='10' style='text-align:center; margin-top: 20px; border-radius: 12px;'>
<tr id='controlPanelSystem'>
<td colspan='3'>Control Panel System</td>
</tr>
<tr align='center' style='color:black;font-weight:bold;font-size:25px'>
<td><input type='button' name='Lock_button' id='Lock_button' value='Lock' onclick='toggle_Button_Lock()' /></td>
<td><input type='button' name='Pump_button' id='Pump_button' value='Pump' onclick='toggle_Button_Pump()' /></td>
<td><input type='button' name='Power_button' id='Power_button' value='Power' onclick='toggle_Button_Power()' /></td>
</tr>
</table>
</div>
<table width='100%' border='3' align='center' bordercolor='#4f6d9f' cellspacing='10' style='text-align:center; margin-top: 20px; border-radius: 12px;'>
<tr id='controlPanelSystem'>
<td colspan='2' id='Temp'> Status </td>
</tr>
<tr align='center' style='color:black;font-weight:bold;font-size:10px'>
<td id='Alarm_Sts'>---</td>
<td id='My_IP'>---</td>
</tr>
<tr align='center' style='color:black;font-weight:bold;font-size:10px'>
<td id='Modem_IP'>---</td>
<td id='My_Mac'>---</td>
</tr>
<tr align='center' style='color:black;font-weight:bold;font-size:10px'>
<td id='SSID_Con'>---</td>
<td id='Internet_Con'>---</td>
</tr>
<tr align='center' style='color:black;font-weight:bold;font-size:10px'>
<td id='Server_Con'>---</td>
<td id='Last_Con'>---</td>
</tr>
<tr align='center' style='color:black;font-weight:bold;font-size:10px'>
<td id='Current_Time'>---</td> <!-- جدید: قسمت نمایش زمان -->
<td id='Current_Date'>---</td> <!-- جدید: قسمت نمایش تاریخ -->
</tr>
</table>
<div class='footer'>
<center>Copyright &copy; 2024 <a style='color:#FFF;' href='http://www.nianelectronic.com'>Nian Electronic Co.</a>Prg-Ver:A1ETP113   </center>
</div>
</div>
</body>
</html>
<script>
function normalizeIP(ip) {return ip.split('.').map(octet => parseInt(octet, 10)).join('.');}
let clientIP = '192.168.001.100';
clientIP = normalizeIP(clientIP);
let isDragging = false;
let progressBar, progressBarFill, autoModeCheckbox;
document.addEventListener('DOMContentLoaded', function() {
    console.log('Client IP:', clientIP);
    progressBar = document.getElementById('progress-bar');
    progressBarFill = document.getElementById('progress-bar-fill');
    autoModeCheckbox = document.getElementById('autoMode');
    progressBar.addEventListener('mousedown', function(e) {if (!autoModeCheckbox.checked) {isDragging = true;updateProgress(e);}});
    progressBar.addEventListener('mousemove', function(e) {if (isDragging && !autoModeCheckbox.checked) {updateProgress(e);}});
    progressBar.addEventListener('mouseup', function() {if (!autoModeCheckbox.checked) {isDragging = false;postProgress();}});
    progressBar.addEventListener('mouseleave', function() {if (!autoModeCheckbox.checked) {isDragging = false;}});
    autoModeCheckbox.addEventListener('change', function() {if (autoModeCheckbox.checked) {progressBar.classList.add('locked');} else { progressBar.classList.remove('locked');}});
    populateTemperatureSelect();
    setTimeout(function() {updateStatus();setInterval(updateStatus, 10000); }, 5000); 
    // فراخوانی تابع دریافت زمان
    fetchGlobalTime();
    setInterval(fetchGlobalTime, 1000);
});
function postData(url = '', data = {}, callback) 
    {var xhr = new XMLHttpRequest();
    xhr.open('POST', url, true);
    xhr.setRequestHeader('Content-Type', 'application/json');
    xhr.onreadystatechange = function(){
        if (xhr.readyState === 4 && xhr.status === 200) 
        {var statusData = JSON.parse(xhr.responseText);
        if (callback) callback(xhr.responseText);} 
        else if (xhr.readyState === 4) {console.error('Error:', xhr.status, xhr.responseText);}};
        xhr.send(JSON.stringify(data));}
function checkIPAndPostData(endpoint, data) {postData(`http://${clientIP}:80/${endpoint}`, data);}
function toggleButton(buttonId) {let buttonElement = document.getElementById(buttonId);let isActive = buttonElement.classList.toggle('toggled');return isActive;}
function toggle_Button_Lock() {let isActive = toggleButton('Lock_button');postData(`http://${clientIP}:80/Update_Parameter`, { 'Lock': isActive.toString() });}
function toggle_Button_Pump() {let isActive = toggleButton('Pump_button');postData(`http://${clientIP}:80/Update_Parameter`, { 'Pump': isActive.toString() });}
function toggle_Button_Power(){let isActive = toggleButton('Power_button');postData(`http://${clientIP}:80/Update_Parameter`, { 'Power': isActive.toString() });}
function handleCheckboxChange(){let isChecked = document.getElementById('autoMode').checked;postData(`http://${clientIP}:80/Update_Parameter`, { 'AutoMode': isChecked.toString() });}
function handleTemperatureChange(){let temperature = document.getElementById('temperatureSelect').value;postData(`http://${clientIP}:80/Update_Parameter`, { 'Temp': temperature.toString() });}
function updateProgress(e){const progressBarRect = progressBar.getBoundingClientRect();const offsetX = e.clientX - progressBarRect.left;const progressPercent = Math.max(0, Math.min(100, (offsetX / progressBarRect.width) * 100));
    progressBarFill.style.width = progressPercent + '%';
    progressBarFill.textContent = Math.round(progressPercent) + '%';}
function postProgress() {const progressValue = parseInt(progressBarFill.textContent, 10);postData(`http://${clientIP}:80/Update_Parameter`, { 'Speed': progressValue.toString() });}
function populateTemperatureSelect() {let select = document.getElementById('temperatureSelect');for (let i = 18; i < 30; i++) {let option = document.createElement('option');option.value = i;option.text = i + '°C';select.add(option);}}
function updateStatus() {
    var xhr = new XMLHttpRequest();
    xhr.open('POST', `http://${clientIP}:80/status`, true);
    xhr.setRequestHeader('Content-Type', 'application/json');
    xhr.onreadystatechange = function() {
        if (xhr.readyState === 4 && xhr.status === 200) {
            var statusData = JSON.parse(xhr.responseText);
            document.getElementById('Alarm_Sts').textContent = statusData.value1;
            document.getElementById('My_IP').textContent = statusData.value2;
            document.getElementById('Modem_IP').textContent = statusData.value3;
            document.getElementById('My_Mac').textContent = statusData.value4;
            document.getElementById('SSID_Con').textContent = statusData.value5;
            document.getElementById('Internet_Con').textContent = statusData.value6;
            document.getElementById('Server_Con').textContent = statusData.value7;
            document.getElementById('Last_Con').textContent = statusData.value8;
            document.getElementById('Temp').textContent = statusData.value9;
            const temperatureSelect = document.getElementById('temperatureSelect');
            const progressBarFill = document.getElementById('progress-bar-fill');
            temperatureSelect.value = statusData.temperature;
            progressBarFill.style.width = statusData.progressBar + '%';
            progressBarFill.textContent = statusData.progressBar + '%';
        }
    };
    xhr.send();
}

// تابع جدید برای دریافت زمان از سرور
async function fetchGlobalTime() {
    const servers = [
        'http://worldtimeapi.org/api/timezone/Asia/Tehran', // سرور اول
        'https://timeapi.io/api/TimeZone/zone?timeZone=Asia/Tehran' // سرور دوم
    ];

    for (const server of servers) {
        try {
            const response = await fetch(server);
            if (!response.ok) {
                throw new Error(`خطا در دریافت زمان از سرور: ${server}`);
            }
            const data = await response.json();

            // استخراج زمان و تاریخ
            let dateTime;
            if (server.includes('worldtimeapi')) {
                dateTime = new Date(data.datetime); // WorldTimeAPI
            } else if (server.includes('timeapi.io')) {
                dateTime = new Date(data.currentLocalTime); // TimeAPI
            }

            const timeString = dateTime.toLocaleTimeString('fa-IR'); // ساعت به فرمت فارسی
            const dateString = dateTime.toLocaleDateString('fa-IR'); // تاریخ به فرمت فارسی

            // نمایش زمان و تاریخ در صفحه
            document.getElementById("Current_Time").textContent = timeString;
            document.getElementById("Current_Date").textContent = dateString;
            return; // اگر موفق بود، از حلقه خارج شو
        } catch (error) {
            console.error(error);
        }
    }

    // اگر همه سرورها ارتباط نداشتن
    document.getElementById("Current_Time").textContent = "خطای اتصال";
    document.getElementById("Current_Date").textContent = "خطای اتصال";
}
</script>
</div>
</body>
</html>