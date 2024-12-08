# docker_mini_project

Project to create simple webpage to check timetable use resources from docker

Guides

**1. Install Docker and Start PostgreSQL in Docker**

```bash
# Pull the latest PostgreSQL Docker image
docker pull postgres:latest

# Create a Docker container for the PostgreSQL database
docker run --name muhammad-db -e POSTGRES_USER=Muhammad -e POSTGRES_PASSWORD=1234 -d -p 5432:5432 postgres:latest


# Verify the container is running
docker ps

# Access the running PostgreSQL container
docker exec -it muhammad-db psql -U Muhammad
```

### 2. **Set Up the PostgreSQL Database and Tables**

```sql

-- Create the Timetable table
CREATE TABLE Timetable (
    course_id SERIAL PRIMARY KEY,
    course_name VARCHAR(255),
    day VARCHAR(50),
    time VARCHAR(50),
    room VARCHAR(50),
    level INT
);

NSERT INTO Timetable (course_name, day, time, room, level) VALUES
    ('Introduction to Political Theory', 'Wednesday', '11:30 AM - 01:50 PM', 'WebNet+', 1),
    ('Computer Programming II', 'Monday', '09:00 AM - 11:20 AM', 'North Hall Classrooms 114', 1),
    ('Public Speaking', 'Friday', '02:00 PM - 04:20 PM', 'North Hall Classrooms 408', 1),
    ('Computer Languages', 'Tuesday', '11:30 AM - 01:50 PM', 'North Hall Classrooms 115', 1),
    ('Operating Systems', 'Monday', '04:30 PM - 06:50 PM', 'North Hall Classrooms 304', 1);

CREATE TABLE Students (
    student_id SERIAL PRIMARY KEY,
    name VARCHAR(255),
    level INT
);
-- Insert sample data into the Students table
INSERT INTO Students (name, level) VALUES
('Muhammad Murodov', 1);

-- Example: Rooms Table (optional if you want to track rooms separately)
CREATE TABLE Rooms (
    room_id SERIAL PRIMARY KEY,
    room_name VARCHAR(50),
    building_name VARCHAR(50)
);

-INSERT INTO Rooms (room_name, building_name) VALUES
('--', 'Webnet+'),
('Room 114', 'North Hall'),
('Room 408', 'North Hall'),
('Room 115', 'North Hall'),
('Room 304', 'North Hall');


```

### 3. **Install Python and Flask Dependencies**

```bash
# Create and activate a virtual environment
python -m venv venv
source venv/bin/activate  # On Windows use venv\Scripts\activate

# Install Flask and pg8000 for database connection
pip install flask pg8000
```

### 4. **Create Flask Application**

```python
from flask import Flask, render_template, request
import pg8000

app = Flask(__name__)

# Home route to get user input for level
@app.route("/", methods=["GET", "POST"])
def index():
    if request.method == "POST":
        level = request.form["level"]
        return render_template("timetable.html", level=level)
    return render_template("index.html")

# Timetable route to fetch timetable data based on the level
@app.route("/timetable", methods=["GET"])
def timetable():
    level = request.args.get("level")

    # Connect to PostgreSQL using pg8000
    conn = pg8000.connect(user="student", password="student_pass", host="localhost", port=5432, database="postgres")
    cur = conn.cursor()

    # Fetch timetable for the selected level
    query = f"SELECT * FROM Timetable WHERE level = {level};"
    cur.execute(query)
    rows = cur.fetchall()

    message = "No data found for this level." if not rows else None
    return render_template("timetable.html", data=rows, message=message)

# Run Flask app
if __name__ == "__main__":
    app.run(debug=True)
```

### 5. **HTML Templates for Flask**

#### **`index.html` (Form for Level Input)**

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Final Project of Muhammad Murodov</title>
  </head>
  <body>
    <h1>University Timetable</h1>
    <form method="POST">
      <label for="level">Enter Level:</label>
      <input type="number" id="level" name="level" required />
      <button type="submit">Submit</button>
    </form>
  </body>
</html>
```

#### **`timetable.html` (Displaying Timetable)**

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Timetable</title>
  </head>
  <body>
    <h2>Timetable for Level {{ level }}</h2>
    {% if data %}
    <table border="1">
      <thead>
        <tr>
          <th>Course ID</th>
          <th>Course Name</th>
          <th>Day</th>
          <th>Time</th>
          <th>Room</th>
        </tr>
      </thead>
      <tbody>
        {% for row in data %}
        <tr>
          <td>{{ row[0] }}</td>
          <td>{{ row[1] }}</td>
          <td>{{ row[2] }}</td>
          <td>{{ row[3] }}</td>
          <td>{{ row[4] }}</td>
        </tr>
        {% endfor %}
      </tbody>
    </table>
    {% else %}
    <p>{{ message }}</p>
    {% endif %}
    <br />
    <a href="/">Go Back</a>
  </body>
</html>
```

### 6. **Running the Flask Application**

```bash
# Make sure you're in the directory with your Flask app
# Run the Flask app
python app.py
```

### 7. **Accessing the Application**

- Navigate to `http://127.0.0.1:5000/` in your web browser to see the application in action
