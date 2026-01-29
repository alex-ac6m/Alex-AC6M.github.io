<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SOTA Spots Table</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 1rem;
            background: #f9fafb;
        }
        h1 {
            text-align: center;
            margin-bottom: 1rem;
        }
        .table-wrapper {
            overflow-x: auto;
            background: white;
            border-radius: 12px;
            box-shadow: 0 2px 8px rgba(0,0,0,0.1);
        }
        table {
            border-collapse: collapse;
            width: 100%;
            min-width: 500px; /* for better layout on narrow screens */
        }
        th, td {
            text-align: left;
            padding: 12px 16px;
        }
        th {
            background: #1e3a8a;
            color: white;
            position: sticky;
            top: 0;
            z-index: 1;
        }
        tr:nth-child(even) td {
            background: #f3f4f6;
        }
        tr:hover td {
            background: #e0f2fe;
        }
        a {
            color: #2563eb;
            text-decoration: none;
            font-weight: 500;
        }
        a:hover {
            text-decoration: underline;
        }
        /* Make text smaller on small screens */
        @media (max-width: 600px) {
            th, td {
                padding: 8px 10px;
                font-size: 14px;
            }
        }
    </style>
</head>
<body>
    <h1>SOTA Spots Table</h1>
    <div class="table-wrapper" id="table-container">Loading...</div>

    <script>
        async function loadTable(limit = 10) {
            // Choose the columns you want to display
           const includedColumns = ["timeStamp", "activatorCallsign", "frequency", "mode", "summitCode", "points", "comments", "activatorName", "summitName"]; // replace with actual keys from your JSON
           const includedColumnsAlias = ["Time", "Call", "Freq", "Mode", "Summit", "Pts", "Comments", "Name", "Summit Name"]; // replace with actual keys from your JSON

            try {
                const result = await fetch(
                    `https://api-db2.sota.org.uk/api/spots/${limit}/all/all/`,
                    { headers: { 'Accept-Encoding': 'gzip, deflate, br, zstd' } }
                );
                const data = await result.json();

                if (!Array.isArray(data) || data.length === 0) {
                    document.getElementById('table-container').textContent = 'No data found.';
                    return;
                }

                const table = document.createElement('table');
                const thead = document.createElement('thead');
                const tbody = document.createElement('tbody');

                // Header row
                const headerRow = document.createElement('tr');
                includedColumnsAlias.forEach(col => {
                    const th = document.createElement('th');
                    th.textContent = col;
                    headerRow.appendChild(th);
                });
                thead.appendChild(headerRow);

                // Data rows
                data.forEach(item => {
                    const row = document.createElement('tr');
                    includedColumns.forEach(col => {
                        const td = document.createElement('td');

                       if (col === "frequency" && item[col] != null) {
                            // Special handling for frequency
                            const link = document.createElement('a');
                            //link.href = (parseFloat(item[col]) * 1).toString(); // or `/subfolder/${freqStr}.html`
                            link.href = "http://websdr1.kfsdr.com:8901/?tune=" + (parseFloat(item[col]) * 1).toString()
			    if (item["mode"] == "CW")
				link.href += "cw";
			    else if (item["mode"] == "SSB")
				link.href += "usb";
			    link.href += "?zoom=4";
                            link.textContent = item[col];
                            td.appendChild(link);
                       } else if (col === "activatorCallsign" && item[col] != null) {
                            // Special handling for Callsign
                            const link = document.createElement('a');
                            link.href = `https://sotl.as/activators/` + item[col];
                            link.textContent = item[col];
                            td.appendChild(link);
                       } else if (col === "timeStamp" && item[col] != null) {
                            td.textContent = (new Date(item[col]).toISOString().slice(11, 16));


                        } else {
                            td.textContent = item[col] ?? "";
                        }

			row.appendChild(td);

                    });
                    tbody.appendChild(row);
                });

                table.appendChild(thead);
                table.appendChild(tbody);

                const container = document.getElementById('table-container');
                container.innerHTML = '';
                container.appendChild(table);

            } catch (error) {
                document.getElementById('table-container').textContent = 'Error loading data.';
                console.error(error);
            }
        }

        loadTable(10);
    </script>
</body>
</html>

