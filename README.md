# Flask Payment System with PIX Integration

This project is a simple Flask application that allows users to create and confirm payments using the Brazilian PIX payment system. It integrates with a SQLite database to store payment information and provides a web interface for users to interact with the payments. The application also supports real-time updates using Flask-SocketIO.

## Features

- **Create PIX Payment**: Users can create a new payment request by specifying a value. The system generates a unique QR code and stores the payment information in the database.
- **QR Code Generation**: After creating the payment, the system generates a QR code that can be scanned by the payer to complete the payment.
- **Payment Confirmation**: Once the payment is completed, a confirmation request can be sent, and the system updates the payment status in the database.
- **Web Interface**: Users can view payment details through a web page, and payment status is updated in real time.
- **WebSocket Support**: The application uses Flask-SocketIO to emit real-time events when a payment is confirmed.

## Requirements

- qrcode==7.4.2
- pillow==10.2.0
- Flask-SocketIO==5.3.6
- Flask-SQLAlchemy==3.1.1
- pytest

To install the required packages, run the following command:

```bash
pip install -r requirements.txt
```

## Installation

1. Clone the repository:

```bash
git clone https://github.com/guifreschi/comunicacao-em-tempo-real-flask
cd payment-system-flask
```

2. Set up the SQLite database:

```bash
python
>>> from app import db
>>> db.create_all()
```

3. Start the application:

```bash
python app.py
```

The app will be available at [http://127.0.0.1:5000](http://127.0.0.1:5000).

## API Endpoints

### `POST /payments/pix`
Creates a new PIX payment. The request body should contain the following JSON data:

```json
{
  "value": 100.0
}
```

Response:

```json
{
  "message": "The payment has been created.",
  "payment": {
    "id": 1,
    "value": 100.0,
    "expiration_date": "2024-12-22T12:30:00",
    "bank_payment_id": "unique_id",
    "qr_code": "path_to_qr_code"
  }
}
```

### `GET /payments/pix/qr_code/<file_name>`
Retrieves the QR code image for the given payment ID. The `file_name` should be the name of the generated QR code.

### `POST /payments/pix/confirmation`
Confirms a payment by providing the `bank_payment_id` and `value` in the request body:

```json
{
  "bank_payment_id": "unique_id",
  "value": 100.0
}
```

Response:

```json
{
  "message": "The payment has been confirmed."
}
```

### `GET /payments/pix/<payment_id>`
Displays a web page with the payment details and QR code (if not paid). If the payment is confirmed, a confirmation message is shown.

## WebSocket Events

- `connect`: Triggered when a client connects to the server.
- `disconnect`: Triggered when a client disconnects from the server.
- `payment-confirmed-{payment_id}`: Emitted when a payment is confirmed.
