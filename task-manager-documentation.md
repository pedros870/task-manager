# Task Manager Project Documentation

## Project Overview
This project is a simple task management web application with a Flask backend and a React frontend. It allows users to create, read, update, and delete tasks.

## Project Structure
```
task-manager/
├── backend/
│   ├── venv/
│   ├── app.py
│   └── tasks.db
├── frontend/
│   ├── node_modules/
│   ├── public/
│   ├── src/
│   │   ├── App.js
│   │   ├── index.js
│   │   └── ...
│   ├── package.json
│   └── package-lock.json
└── README.md
```

## Backend (Flask)

### File: backend/app.py

```python
# Import necessary libraries
from flask import Flask, request, jsonify
from flask_cors import CORS
from flask_sqlalchemy import SQLAlchemy

# Initialize Flask app
app = Flask(__name__)
# Enable CORS for all routes
CORS(app)
# Configure SQLite database
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///tasks.db'
db = SQLAlchemy(app)

# Define Task model
class Task(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(100), nullable=False)
    completed = db.Column(db.Boolean, default=False)

# Route to get all tasks
@app.route('/tasks', methods=['GET'])
def get_tasks():
    tasks = Task.query.all()
    return jsonify([{'id': task.id, 'title': task.title, 'completed': task.completed} for task in tasks])

# Route to add a new task
@app.route('/tasks', methods=['POST'])
def add_task():
    data = request.json
    new_task = Task(title=data['title'])
    db.session.add(new_task)
    db.session.commit()
    return jsonify({'id': new_task.id, 'title': new_task.title, 'completed': new_task.completed}), 201

# Route to update a task
@app.route('/tasks/<int:task_id>', methods=['PUT'])
def update_task(task_id):
    task = Task.query.get_or_404(task_id)
    data = request.json
    task.title = data.get('title', task.title)
    task.completed = data.get('completed', task.completed)
    db.session.commit()
    return jsonify({'id': task.id, 'title': task.title, 'completed': task.completed})

# Route to delete a task
@app.route('/tasks/<int:task_id>', methods=['DELETE'])
def delete_task(task_id):
    task = Task.query.get_or_404(task_id)
    db.session.delete(task)
    db.session.commit()
    return '', 204

# Run the app
if __name__ == '__main__':
    with app.app_context():
        db.create_all()
    app.run(debug=True)
```

### Dependencies
- Flask==2.3.2
- Flask-CORS==3.0.10
- Flask-SQLAlchemy==3.0.3

## Frontend (React)

### File: frontend/src/App.js

```javascript
import React, { useState, useEffect } from 'react';
import axios from 'axios';

function App() {
  // State for tasks and new task input
  const [tasks, setTasks] = useState([]);
  const [newTask, setNewTask] = useState('');

  // Fetch tasks on component mount
  useEffect(() => {
    fetchTasks();
  }, []);

  // Function to fetch all tasks
  const fetchTasks = async () => {
    const response = await axios.get('http://localhost:5000/tasks');
    setTasks(response.data);
  };

  // Function to add a new task
  const addTask = async (e) => {
    e.preventDefault();
    await axios.post('http://localhost:5000/tasks', { title: newTask });
    setNewTask('');
    fetchTasks();
  };

  // Function to toggle task completion status
  const toggleTask = async (id, completed) => {
    await axios.put(`http://localhost:5000/tasks/${id}`, { completed: !completed });
    fetchTasks();
  };

  // Function to delete a task
  const deleteTask = async (id) => {
    await axios.delete(`http://localhost:5000/tasks/${id}`);
    fetchTasks();
  };

  return (
    <div className="App">
      <h1>Task Manager</h1>
      {/* Form to add new tasks */}
      <form onSubmit={addTask}>
        <input
          type="text"
          value={newTask}
          onChange={(e) => setNewTask(e.target.value)}
          placeholder="Add a new task"
        />
        <button type="submit">Add Task</button>
      </form>
      {/* List of tasks */}
      <ul>
        {tasks.map((task) => (
          <li key={task.id}>
            <input
              type="checkbox"
              checked={task.completed}
              onChange={() => toggleTask(task.id, task.completed)}
            />
            <span style={{ textDecoration: task.completed ? 'line-through' : 'none' }}>
              {task.title}
            </span>
            <button onClick={() => deleteTask(task.id)}>Delete</button>
          </li>
        ))}
      </ul>
    </div>
  );
}

export default App;
```

### Dependencies
- react: 18.2.0
- react-dom: 18.2.0
- axios: 1.4.0

## Setup and Running the Application

### Backend Setup
1. Navigate to the `backend` folder
2. Create a virtual environment: `python -m venv venv`
3. Activate the virtual environment:
   - Windows: `venv\Scripts\activate`
   - macOS/Linux: `source venv/bin/activate`
4. Install dependencies: `pip install flask flask-sqlalchemy flask-cors`
5. Run the Flask app: `python app.py`

### Frontend Setup
1. Navigate to the `frontend` folder
2. Install dependencies: `npm install`
3. Start the React app: `npm start`

## API Endpoints

- GET /tasks: Retrieve all tasks
- POST /tasks: Create a new task
- PUT /tasks/<id>: Update a task
- DELETE /tasks/<id>: Delete a task

## Future Improvements

1. Add user authentication
2. Implement task categories or tags
3. Add due dates for tasks
4. Improve UI/UX with CSS styling
5. Implement error handling and loading states
6. Add unit and integration tests

## Version Information

- Python: 3.8+
- Node.js: 14+
- React: 18.2.0
- Flask: 2.3.2
- SQLAlchemy: 3.0.3

This documentation provides an overview of the Task Manager project, including its structure, main components, setup instructions, and potential future improvements. As you continue to develop the project, remember to update this documentation to reflect any changes or new features.
