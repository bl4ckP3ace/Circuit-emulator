<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Custom Circuit & Safety Simulator</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background: #1e1e1e;
            color: #fff;
            margin: 0;
            display: flex;
            height: 100vh;
            overflow: hidden;
        }
        #sidebar {
            width: 280px;
            background: #252526;
            padding: 20px;
            box-sizing: border-box;
            border-right: 1px solid #333;
            display: flex;
            flex-direction: column;
            gap: 15px;
        }
        h2 { font-size: 18px; margin-top: 0; color: #4CAF50; }
        .btn {
            background: #0e639c;
            color: white;
            border: none;
            padding: 10px;
            border-radius: 4px;
            cursor: pointer;
            font-weight: bold;
            text-align: left;
        }
        .btn:hover { background: #1177bb; }
        #canvas-container {
            flex-grow: 1;
            position: relative;
            background: #121212;
            overflow: hidden;
        }
        canvas { display: block; }
        #console {
            position: absolute;
            bottom: 0;
            left: 280px;
            right: 0;
            height: 150px;
            background: rgba(0, 0, 0, 0.85);
            border-top: 1px solid #333;
            padding: 10px;
            box-sizing: border-box;
            font-family: monospace;
            overflow-y: auto;
            font-size: 13px;
        }
        .log-info { color: #4CAF50; }
        .log-warn { color: #FF9800; }
        .log-error { color: #F44336; font-weight: bold; }
    </style>
</head>
<body>

    <div id="sidebar">
        <h2>🛠️ Components</h2>
        <p style="font-size: 12px; color: #aaa;">Click buttons to spawn components onto the workspace. Drag them around.</p>
        <button class="btn" onclick="addComponent('battery')">🔋 Add Battery (5V)</button>
        <button class="btn" onclick="addComponent('resistor')">⚡ Add Resistor (220Ω)</button>
        <button class="btn" onclick="addComponent('led')">💡 Add LED (Max 20mA)</button>
        <hr style="border-color: #444; width:100%;">
        <button class="btn" style="background: #2ea44f;" onclick="runSimulation()">▶ Run Simulation & Check Safety</button>
        <button class="btn" style="background: #d73a49;" onclick="clearCanvas()">🗑️ Clear Workspace</button>
    </div>

    <div id="canvas-container">
        <canvas id="simCanvas"></canvas>
    </div>

    <div id="console">
        <div class="log-info">[System] Simulator initialized. Ready to build.</div>
    </div>

    <script>
        const canvas = document.getElementById('simCanvas');
        const ctx = canvas.getContext('2d');
        const consoleLog = document.getElementById('console');

        function resizeCanvas() {
            canvas.width = document.getElementById('canvas-container').clientWidth;
            canvas.height = document.getElementById('canvas-container').clientHeight - 150;
        }
        window.addEventListener('resize', resizeCanvas);
        resizeCanvas();

        let components = [];
        let selectedComponent = null;
        let isDragging = false;
        let offsetX, offsetY;

        function log(message, type = 'info') {
            const div = document.createElement('div');
            div.className = `log-${type}`;
            div.innerText = `[${new Date().toLocaleTimeString()}] ${message}`;
            consoleLog.appendChild(div);
            consoleLog.scrollTop = consoleLog.scrollHeight;
        }

        function addComponent(type) {
            let comp = {
                id: type + '_' + Math.random().toString(36.substring(2, 7)),
                type: type,
                x: 100 + Math.random() * 200,
                y: 100 + Math.random() * 200,
                width: 70,
                height: 50,
                burnedOut: false
            };
            components.push(comp);
            log(`Added ${type} to canvas.`);
            draw();
        }

        function draw() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            // Draw grid lines
            ctx.strokeStyle = '#222';
            ctx.lineWidth = 1;
            for (let x = 0; x < canvas.width; x += 30) {
                ctx.beginPath(); ctx.moveTo(x, 0); ctx.lineTo(x, canvas.height); ctx.stroke();
            }
            for (let y = 0; y < canvas.height; y += 30) {
                ctx.beginPath(); ctx.moveTo(0, y); ctx.lineTo(canvas.width, y); ctx.stroke();
            }

            // Draw components
            components.forEach(comp => {
                ctx.fillStyle = comp.burnedOut ? '#550000' : '#333';
                ctx.strokeStyle = comp.burnedOut ? '#ff0000' : '#4CAF50';
                ctx.lineWidth = 2;
                ctx.fillRect(comp.x, comp.y, comp.width, comp.height);
                ctx.strokeRect(comp.x, comp.y, comp.width, comp.height);

                ctx.fillStyle = '#fff';
                ctx.font = '12px Arial';
                let label = comp.type.toUpperCase();
                if (comp.burnedOut) label += " (BURNED!)";
                ctx.fillText(label, comp.x + 8, comp.y + 30);
            });
        }

        // Mouse interactions for dragging components
        canvas.addEventListener('mousedown', (e) => {
            const rect = canvas.getBoundingClientRect();
            const mouseX = e.clientX - rect.left;
            const mouseY = e.clientY - rect.top;

            selectedComponent = components.reverse().find(c => 
                mouseX >= c.x && mouseX <= c.x + c.width &&
                mouseY >= c.y && mouseY <= c.y + c.height
            );
            components.reverse();

            if (selectedComponent) {
                isDragging = true;
                offsetX = mouseX - selectedComponent.x;
                offsetY = mouseY - selectedComponent.y;
            }
        });

        canvas.addEventListener('mousemove', (e) => {
            if (isDragging && selectedComponent) {
                const rect = canvas.getBoundingClientRect();
                selectedComponent.x = (e.clientX - rect.left) - offsetX;
                selectedComponent.y = (e.clientY - rect.top) - offsetY;
                draw();
            }
        });

        window.addEventListener('mouseup', () => { isDragging = false; });

        function clearCanvas() {
            components = [];
            log("Workspace cleared.", "info");
            draw();
        }

        // Simulation & Safety Inspection Engine
        function runSimulation() {
            log("Running circuit analysis and safety check...", "info");

            let hasBattery = components.some(c => c.type === 'battery');
            let hasLed = components.some(c => c.type === 'led');
            let hasResistor = components.some(c => c.type === 'resistor');

            if (!hasBattery) {
                log("Simulation Error: No power source (Battery) detected in the circuit.", "error");
                return;
            }
            if (!hasLed) {
                log("Simulation Warning: Circuit is incomplete or missing output loads (LED).", "warn");
                return;
            }

            // Physics calculation: Ohm's Law (Assuming 5V source, 2000mV LED forward voltage drop)
            const supplyVoltage = 5.0;
            const ledForwardVoltage = 2.0;
            
            if (!hasResistor) {
                // Without a resistor, direct connection causes overcurrent!
                components.forEach(c => {
                    if (c.type === 'led') c.burnedOut = true;
                });
                draw();
                log("🔥 CRITICAL SAFETY FAULT: Direct short / Missing current-limiting resistor!", "error");
                log("💥 LED destroyed! Current exceeded maximum safe threshold (>100mA).", "error");
            } else {
                // With a 220 ohm resistor: I = (5V - 2V) / 220Ω = 13.6mA
                let resistance = 220; 
                let currentMa = ((supplyVoltage - ledForwardVoltage) / resistance) * 1000;

                components.forEach(c => { if (c.type === 'led') c.burnedOut = false; });
                draw();

                log(`✅ Circuit operating safely. Calculated LED Current: ${currentMa.toFixed(1)}mA (Max limit: 20mA).`, "info");
                log("✨ Simulation status: Stable. No warnings detected.", "info");
            }
        }
    </script>
</body>
</html>

