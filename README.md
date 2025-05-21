# Chattr
Communication app
import React, { useState, useEffect, useRef } from "react";
import { Input } from "@/components/ui/input";
import { Button } from "@/components/ui/button";
import { Card, CardContent } from "@/components/ui/card";
import { io } from "socket.io-client";

const socket = io("http://localhost:3001"); // Adjust to your server URL

const CommunicationApp = () => {
  const [messages, setMessages] = useState([]);
  const [input, setInput] = useState("");
  const messagesEndRef = useRef(null);

  useEffect(() => {
    socket.on("receive_message", (message) => {
      setMessages((prevMessages) => [...prevMessages, message]);
    });

    return () => {
      socket.off("receive_message");
    };
  }, []);

  useEffect(() => {
    messagesEndRef.current?.scrollIntoView({ behavior: "smooth" });
  }, [messages]);

  const sendMessage = () => {
    if (input.trim() === "") return;
    const newMessage = {
      id: Date.now(),
      text: input,
      sender: "You",
    };
    setMessages([...messages, newMessage]);
    socket.emit("send_message", newMessage);
    setInput("");
  };

  return (
    <div className="max-w-xl mx-auto p-4 space-y-4">
      <h1 className="text-2xl font-bold">Real-Time Communication App</h1>
      <Card className="h-80 overflow-y-auto p-2 bg-gray-50">
        <CardContent className="space-y-2">
          {messages.map((msg) => (
            <div
              key={msg.id}
              className="bg-white shadow p-2 rounded-xl border border-gray-200"
            >
              <strong>{msg.sender}:</strong> {msg.text}
            </div>
          ))}
          <div ref={messagesEndRef} />
        </CardContent>
      </Card>
      <div className="flex space-x-2">
        <Input
          placeholder="Type your message..."
          value={input}
          onChange={(e) => setInput(e.target.value)}
          onKeyDown={(e) => e.key === "Enter" && sendMessage()}
        />
        <Button onClick={sendMessage}>Send</Button>
      </div>
    </div>
  );
};

export default CommunicationApp;
