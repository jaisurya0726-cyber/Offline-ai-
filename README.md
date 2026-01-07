
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Offline AI Assistant</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            height: 100vh;
            display: flex;
            align-items: center;
            justify-content: center;
        }

        .chat-container {
            width: 90%;
            max-width: 800px;
            height: 90vh;
            background: rgba(255, 255, 255, 0.95);
            backdrop-filter: blur(10px);
            border-radius: 20px;
            box-shadow: 0 20px 40px rgba(0, 0, 0, 0.1);
            display: flex;
            flex-direction: column;
            overflow: hidden;
        }

        .header {
            background: linear-gradient(135deg, #4facfe 0%, #00f2fe 100%);
            padding: 20px;
            text-align: center;
            color: white;
        }

        .header h1 {
            font-size: 1.5em;
            font-weight: 600;
        }

        .status {
            font-size: 0.9em;
            opacity: 0.9;
            margin-top: 5px;
        }

        .chat-messages {
            flex: 1;
            padding: 20px;
            overflow-y: auto;
            scroll-behavior: smooth;
        }

        .message {
            margin-bottom: 20px;
            display: flex;
            animation: fadeInUp 0.3s ease-out;
        }

        .message.user {
            justify-content: flex-end;
        }

        .message.assistant {
            justify-content: flex-start;
        }

        .message-content {
            max-width: 70%;
            padding: 12px 16px;
            border-radius: 18px;
            font-size: 0.95em;
            line-height: 1.4;
        }

        .message.user .message-content {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
        }

        .message.assistant .message-content {
            background: #f1f3f5;
            color: #333;
            border: 1px solid #e9ecef;
        }

        .typing-indicator {
            display: none;
            padding: 12px 16px;
            background: #f1f3f5;
            border-radius: 18px;
            margin-bottom: 20px;
        }

        .typing-dots {
            display: flex;
            align-items: center;
        }

        .typing-dots span {
            height: 8px;
            width: 8px;
            border-radius: 50%;
            background: #999;
            margin: 0 2px;
            animation: typing 1.4s infinite ease-in-out;
        }

        .typing-dots span:nth-child(1) { animation-delay: -0.32s; }
        .typing-dots span:nth-child(2) { animation-delay: -0.16s; }

        .input-container {
            padding: 20px;
            background: white;
            border-top: 1px solid #e9ecef;
        }

        .input-form {
            display: flex;
            gap: 10px;
        }

        .message-input {
            flex: 1;
            padding: 12px 16px;
            border: 2px solid #e9ecef;
            border-radius: 25px;
            font-size: 0.95em;
            outline: none;
            transition: border-color 0.3s ease;
        }

        .message-input:focus {
            border-color: #667eea;
        }

        .send-button {
            padding: 12px 20px;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            border: none;
            border-radius: 25px;
            cursor: pointer;
            font-weight: 600;
            transition: transform 0.2s ease;
        }

        .send-button:hover:not(:disabled) {
            transform: translateY(-2px);
        }

        .send-button:disabled {
            opacity: 0.6;
            cursor: not-allowed;
        }

        @keyframes fadeInUp {
            from {
                opacity: 0;
                transform: translateY(20px);
            }
            to {
                opacity: 1;
                transform: translateY(0);
            }
        }

        @keyframes typing {
            0%, 60%, 100% {
                transform: translateY(0);
            }
            30% {
                transform: translateY(-10px);
            }
        }

        .welcome-message {
            text-align: center;
            color: #666;
            font-style: italic;
            margin: 40px 0;
        }

        @media (max-width: 768px) {
            .chat-container {
                width: 95%;
                height: 95vh;
                border-radius: 15px;
            }
            
            .message-content {
                max-width: 85%;
            }
        }
    </style>
</head>
<body>
    <div class="chat-container">
        <div class="header">
            <h1>🤖 Offline AI Assistant</h1>
            <div class="status">✅ Ready to help • No internet required</div>
        </div>

        <div class="chat-messages" id="chatMessages">
            <div class="welcome-message">
                👋 Hello! I'm your offline AI assistant. I can help with general questions, calculations, writing tasks, and more!
            </div>
        </div>

        <div class="typing-indicator" id="typingIndicator">
            <div class="typing-dots">
                <span></span>
                <span></span>
                <span></span>
            </div>
        </div>

        <div class="input-container">
            <form class="input-form" id="messageForm">
                <input 
                    type="text" 
                    class="message-input" 
                    id="messageInput" 
                    placeholder="Ask me anything..."
                    autocomplete="off"
                >
                <button type="submit" class="send-button" id="sendButton">Send</button>
            </form>
        </div>
    </div>

    <script>
        class OfflineAI {
            constructor() {
                this.responses = {
                    greetings: [
                        "Hello! How can I help you today?",
                        "Hi there! What can I assist you with?",
                        "Greetings! I'm here to help.",
                        "Hey! What would you like to know?"
                    ],
                    thanks: [
                        "You're welcome! Happy to help!",
                        "Glad I could assist you!",
                        "No problem at all!",
                        "You're very welcome!"
                    ],
                    calculations: {
                        pattern: /(\d+\.?\d*)\s*([+\-*/])\s*(\d+\.?\d*)/,
                        handler: (match) => {
                            const [, num1, operator, num2] = match;
                            const a = parseFloat(num1);
                            const b = parseFloat(num2);
                            let result;
                            
                            switch(operator) {
                                case '+': result = a + b; break;
                                case '-': result = a - b; break;
                                case '*': result = a * b; break;
                                case '/': result = b !== 0 ? a / b : 'Cannot divide by zero'; break;
                                default: return null;
                            }
                            
                            return `${num1} ${operator} ${num2} = ${result}`;
                        }
                    },
                    time: () => {
                        const now = new Date();
                        return `Current time: ${now.toLocaleTimeString()}\nCurrent date: ${now.toLocaleDateString()}`;
                    },
                    weather: [
                        "I'm an offline assistant, so I can't check current weather. Try looking outside or checking a weather app!",
                        "Since I'm offline, I can't access live weather data. Check your local weather service for current conditions."
                    ],
                    knowledge: {
                        'what is javascript': 'JavaScript is a high-level, interpreted programming language used primarily for web development to create interactive websites.',
                        'what is html': 'HTML (HyperText Markup Language) is the standard markup language used to create web pages and web applications.',
                        'what is css': 'CSS (Cascading Style Sheets) is a style sheet language used to describe the presentation and formatting of HTML documents.',
                        'what is python': 'Python is a high-level, general-purpose programming language known for its simple syntax and versatility.',
                        'how to learn programming': 'Start with the basics: choose a language (Python is beginner-friendly), practice regularly, build projects, and use online resources like tutorials and documentation.',
                        'what is ai': 'AI (Artificial Intelligence) refers to computer systems that can perform tasks typically requiring human intelligence, such as learning, reasoning, and problem-solving.'
                    },
                    default: [
                        "That's an interesting question! While I'm an offline assistant with limited knowledge, I'll do my best to help.",
                        "I'm not sure about that specific topic, but I can help with general questions, calculations, and basic programming concepts.",
                        "As an offline AI, my knowledge is limited, but I'm happy to assist with what I can!",
                        "I don't have information about that particular topic, but feel free to ask me about other things!"
                    ]
                };
            }

            generateResponse(message) {
                const msg = message.toLowerCase().trim();
                
                // Greetings
                if (msg.match(/^(hi|hello|hey|greetings|good morning|good afternoon|good evening)$/)) {
                    return this.getRandomResponse(this.responses.greetings);
                }
                
                // Thanks
                if (msg.match(/(thank|thanks|thx)/)) {
                    return this.getRandomResponse(this.responses.thanks);
                }
                
                // Time/Date
                if (msg.match(/(time|date|clock)/)) {
                    return this.responses.time();
                }
                
                // Weather
                if (msg.match(/weather/)) {
                    return this.getRandomResponse(this.responses.weather);
                }
                
                // Calculations
                const calcMatch = msg.match(this.responses.calculations.pattern);
                if (calcMatch) {
                    return this.responses.calculations.handler(calcMatch);
                }
                
                // Knowledge base
                for (const [key, value] of Object.entries(this.responses.knowledge)) {
                    if (msg.includes(key)) {
                        return value;
                    }
                }
                
                // Name questions
                if (msg.match(/(what.*name|who are you|what are you)/)) {
                    return "I'm your offline AI assistant! I'm here to help with questions, calculations, and general information.";
                }
                
                // Capabilities
                if (msg.match(/(what can you do|help|capabilities)/)) {
                    return "I can help with:\n• Basic calculations\n• General knowledge questions\n• Programming concepts\n• Current time and date\n• Simple conversations\n\nJust ask me anything!";
                }
                
                return this.getRandomResponse(this.responses.default);
            }
            
            getRandomResponse(responses) {
                return responses[Math.floor(Math.random() * responses.length)];
            }
        }

        class ChatInterface {
            constructor() {
                this.ai = new OfflineAI();
                this.chatMessages = document.getElementById('chatMessages');
                this.messageInput = document.getElementById('messageInput');
                this.sendButton = document.getElementById('sendButton');
                this.messageForm = document.getElementById('messageForm');
                this.typingIndicator = document.getElementById('typingIndicator');
                
                this.init();
            }
            
            init() {
                this.messageForm.addEventListener('submit', (e) => this.handleSubmit(e));
                this.messageInput.addEventListener('keydown', (e) => {
                    if (e.key === 'Enter' && !e.shiftKey) {
                        e.preventDefault();
                        this.handleSubmit(e);
                    }
                });
            }
            
            async handleSubmit(e) {
                e.preventDefault();
                const message = this.messageInput.value.trim();
                
                if (!message) return;
                
                this.addMessage(message, 'user');
                this.messageInput.value = '';
                this.sendButton.disabled = true;
                
                this.showTyping();
                
                // Simulate AI thinking time
                await this.delay(1000 + Math.random() * 2000);
                
                const response = this.ai.generateResponse(message);
                this.hideTyping();
                this.addMessage(response, 'assistant');
                this.sendButton.disabled = false;
                this.messageInput.focus();
            }
            
            addMessage(content, sender) {
                const messageDiv = document.createElement('div');
                messageDiv.className = `message ${sender}`;
                
                const contentDiv = document.createElement('div');
                contentDiv.className = 'message-content';
                contentDiv.textContent = content;
                
                messageDiv.appendChild(contentDiv);
                this.chatMessages.appendChild(messageDiv);
                
                this.scrollToBottom();
            }
            
            showTyping() {
                this.typingIndicator.style.display = 'block';
                this.scrollToBottom();
            }
            
            hideTyping() {
                this.typingIndicator.style.display = 'none';
            }
            
            scrollToBottom() {
                this.chatMessages.scrollTop = this.chatMessages.scrollHeight;
            }
            
            delay(ms) {
                return new Promise(resolve => setTimeout(resolve, ms));
            }
        }
        
        // Initialize the chat interface when page loads
        document.addEventListener('DOMContentLoaded', () => {
            new ChatInterface();
        });
    </script>
</body>
</html>