# Application-Health-Checker-Bash
# System Health Monitoring & Application Health Checker
This script continuously monitors CPU, memory, and disk usage and alerts when any of these exceed predefined thresholds (default: 80%). It extracts system resource usage using built-in Linux commands and compares them against the set limits. If a threshold is crossed, the script displays a warning message. The monitoring runs indefinitely, checking every 5 seconds.

The script also includes an Application Health Checker that verifies whether a web application is running by sending an HTTP request. It checks the status code of the response—if it returns 200, the application is UP; otherwise, it is DOWN. The user needs to specify the application’s URL while running the script.

To use the script, make it executable and run it in a Linux terminal. The system monitor will display real-time resource usage, and the application checker will report if the service is accessible.

# System Health Monitoring Section
This part continuously checks the system's resource usage.

bash
CPU_THRESHOLD=80
MEMORY_THRESHOLD=80
DISK_THRESHOLD=80

These are the thresholds (in percentage) that trigger an alert.
If CPU, memory, or disk usage exceeds the threshold, the script displays an alert.

bash
while true; do
    CPU_USAGE=$(top -bn1 | grep "Cpu(s)" | awk '{print $2 + $4}')
    MEMORY_USAGE=$(free | awk '/Mem/{printf("%.2f"), $3/$2*100}')
    DISK_USAGE=$(df / | awk 'NR==2 {print $5}' | sed 's/%//')

    top -bn1 | grep "Cpu(s)" | awk '{print $2 + $4}' → Extracts CPU usage.

  free | awk '/Mem/{printf("%.2f"), $3/$2*100}' → Extracts memory usage.

   df / | awk 'NR==2 {print $5}' | sed 's/%//' → Extracts disk usage percentage.

bash
echo "CPU Usage: $CPU_USAGE% | Memory Usage: $MEMORY_USAGE% | Disk Usage: $DISK_USAGE%"

Displays current system usage.
bash
    if (( $(echo "$CPU_USAGE > $CPU_THRESHOLD" | bc -l) )); then
        echo "⚠️ ALERT: CPU usage exceeded threshold!"
    fi
    
    if (( $(echo "$MEMORY_USAGE > $MEMORY_THRESHOLD" | bc -l) )); then
        echo "⚠️ ALERT: Memory usage exceeded threshold!"
    fi
    
    if (( DISK_USAGE > DISK_THRESHOLD )); then
        echo "⚠️ ALERT: Disk usage exceeded threshold!"
    fi

    sleep 5  # Check every 5 seconds
done

Compares system usage with the thresholds.

If any metric exceeds the threshold, an alert is displayed.

The script runs continuously, checking every 5 seconds.

Application Health Checker Section

This part checks if a web application is running.

bash
check_application() {
    URL="$1"
    RESPONSE=$(curl -o /dev/null -s -w "%{http_code}" "$URL")

    curl -o /dev/null -s -w "%{http_code}" "$URL" → Sends a request to the given URL and extracts the HTTP status code.

        if [ "$RESPONSE" -eq 200 ]; then
        echo "✅ Application is UP! Status Code: $RESPONSE"
    else
        echo "❌ Application is DOWN! Status Code: $RESPONSE"
    fi
}
If the status code is 200, the app is UP.

Otherwise, the app is DOWN.

# Example usage of Application Health Checker
check_application "http://your-application-url.com"
