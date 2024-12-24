# Flask Pix Payment System

A study project using Flask to simulate a Pix payment system with QR code generation, WebSocket notifications, and MySQL integration. This project features a pre-built frontend for improved user interaction.

## Features
- Pix payment creation with QR code generation.
- Payment confirmation via WebSocket notifications.
- MySQL database integration for managing payment records.
- Web routes to display payment and confirmation pages.

## Technologies Used
- **Flask**: Backend framework.
- **Flask-SocketIO**: Real-time communication with WebSockets.
- **MySQL**: Database for storing payment information.
- **HTML/CSS**: Pre-built frontend templates.

## Prerequisites
- qrcode==7.4.2
- pillow==10.2.0
- Flask-SocketIO==5.3.6
- Flask-SQLAlchemy==3.1.1
- pytest
- pymysql==1.1.0

## Setup Instructions

1. Clone the repository:  
   ```bash
   git clone https://github.com/guifreschi/comunicacao-em-tempo-real-flask.git
   cd comunicacao-em-tempo-real-flask
   ```

2. Build and start the services using Docker Compose:  
   ```bash
   docker-compose up --build
   ```

   This will start the MySQL database container with the configured database and user.

3. Access the MySQL database:  
   Use a MySQL client or the VSCode MySQL extension with the following settings:  
   - **Host:** `127.0.0.1`  
   - **Port:** `3306`  
   - **Username:** `admin`  
   - **Password:** `admin123`  
   - **Database:** `flask-socketio-db`  

4. Update the Flask configuration:  
   Ensure that the `app.py` file contains the following connection string in `app.config['SQLALCHEMY_DATABASE_URI']`:
   ```python
   app.config['SQLALCHEMY_DATABASE_URI'] = 'mysql+pymysql://root:admin123@127.0.0.1:3306/flask-socketio-db'
   ```

5. Initialize the database:  
   If not automated, you can initialize the database schema manually by accessing the Flask container:
   ```bash
   docker exec -it <flask-container-name> flask db upgrade
   ```

6. Run the application:  
   After starting the Docker containers, the Flask application will be accessible at:
   ```bash
   http://127.0.0.1:5000
   ```

7. Access logs (optional):  
   To debug or monitor the application, view the logs with:
   ```bash
   docker-compose logs -f
   ```

## Code Overview

### `Pix` Class
Handles Pix payment creation, including QR code generation.
```python
import uuid
import qrcode

class Pix:
  def __init__(self):
    pass

  def create_payment(self, base_dir=""):
    # Create the payment at the financial institution
    bank_payment_id = str(uuid.uuid4())

    # Copy and paste code
    hash_payment = f'hash_payment_{bank_payment_id}'

    # Generate QR code
    img = qrcode.make(hash_payment)

    # Save the QR code image
    img.save(f"{base_dir}static/img/qr_code_payment_{bank_payment_id}.png")

    return {
      "bank_payment_id": bank_payment_id,
      "qr_code_path": f"qr_code_payment_{bank_payment_id}"
    }
```

### `Payment` Database Model
Represents a payment record in the MySQL database.
```python
from repository.database import db

class Payment(db.Model):
  id = db.Column(db.Integer, primary_key=True)
  value = db.Column(db.Float)
  paid = db.Column(db.Boolean, default=False)
  bank_payment_id = db.Column(db.String(200), nullable=True)
  qr_code = db.Column(db.String(100), nullable=True)
  expiration_date = db.Column(db.DateTime)

  def to_dict(self):
    return {
      "id": self.id,
      "value": self.value,
      "paid": self.paid,
      "bank_payment_id": self.bank_payment_id,
      "qr_code": self.qr_code,
      "expiration_date": self.expiration_date
    }
```

## API Endpoints

### Create a Pix Payment
**POST** `/payments/pix`
- **Request Body:**
  ```json
  {
    "value": 100.00
  }
  ```
- **Response:**
  ```json
  {
    "message": "The payment has been created.",
    "payment": {
      "id": 1,
      "value": 100.0,
      "qr_code": "static/img/qrcode.png",
      "expiration_date": "2024-12-31T12:00:00"
    }
  }
  ```

### Confirm a Pix Payment
**POST** `/payments/pix/confirmation`
- **Request Body:**
  ```json
  {
    "bank_payment_id": "123456",
    "value": 100.00
  }
  ```
- **Response:**
  ```json
  {
    "message": "The payment has been confirmed."
  }
  ```

### Get Payment Page
**GET** `/payments/pix/<payment_id>`
- Displays the payment page with QR code or the confirmation page if already paid.

## WebSocket Events
- **Connect:** Logs when a client connects.
- **Disconnect:** Logs when a client disconnects.
- **Payment Confirmation:** Emits `payment-confirmed-{payment_id}` upon payment confirmation.

## Author
Developed by [Guilherme Freschi](https://github.com/guifreschi).

