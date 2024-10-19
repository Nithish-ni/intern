# intern
intern
#TASK 1

Back end
CREATE TABLE Registration (
    ID INT PRIMARY KEY AUTO_INCREMENT,
    Name VARCHAR(100) NOT NULL,
    Email VARCHAR(100) NOT NULL UNIQUE,
    DateOfBirth DATE NOT NULL,
    PhoneNumber VARCHAR(15),
    Address VARCHAR(255),
    RegistrationDate TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    IsActive BOOLEAN DEFAULT TRUE,
    CONSTRAINT chk_age CHECK (DATEDIFF(CURDATE(), DateOfBirth) / 365 >= 18) -- Ensure user is at least 18 years old
);


#TASK2


Task2
pip install Flask SQLAlchemy
DATABASE MODEL
from flask_sqlalchemy import SQLAlchemy
from datetime import datetime

db = SQLAlchemy()

class Registration(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(100), nullable=False)
    email = db.Column(db.String(100), unique=True, nullable=False)
    date_of_birth = db.Column(db.Date, nullable=False)
    phone_number = db.Column(db.String(15))
    address = db.Column(db.String(255))
    registration_date = db.Column(db.DateTime, default=datetime.utcnow)
    is_active = db.Column(db.Boolean, default=True)
CRUD FUNCTIONS
from flask import Flask, request, jsonify
from flask_sqlalchemy import SQLAlchemy
from models import db, Registration
from datetime import datetime

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///registrations.db'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
db.init_app(app)

with app.app_context():
    db.create_all()

# Create
@app.route('/registrations', methods=['POST'])
def create_registration():
    data = request.json
    try:
        new_registration = Registration(
            name=data['name'],
            email=data['email'],
            date_of_birth=data['date_of_birth'],
            phone_number=data.get('phone_number'),
            address=data.get('address')
        )
        db.session.add(new_registration)
        db.session.commit()
        return jsonify({'id': new_registration.id}), 201
    except Exception as e:
        db.session.rollback()
        return jsonify({'error': str(e)}), 400

# Read
@app.route('/registrations', methods=['GET'])
def get_registrations():
    registrations = Registration.query.all()
    return jsonify([{
        'id': r.id,
        'name': r.name,
        'email': r.email,
        'date_of_birth': r.date_of_birth,
        'phone_number': r.phone_number,
        'address': r.address,
        'registration_date': r.registration_date,
        'is_active': r.is_active
    } for r in registrations]), 200

@app.route('/registrations/<int:id>', methods=['GET'])
def get_registration(id):
    registration = Registration.query.get(id)
    if registration:
        return jsonify({
            'id': registration.id,
            'name': registration.name,
            'email': registration.email,
            'date_of_birth': registration.date_of_birth,
            'phone_number': registration.phone_number,
            'address': registration.address,
            'registration_date': registration.registration_date,
            'is_active': registration.is_active
        }), 200
    return jsonify({'error': 'Registration not found'}), 404

# Update
@app.route('/registrations/<int:id>', methods=['PUT'])
def update_registration(id):
    data = request.json
    registration = Registration.query.get(id)
    if registration:
        try:
            registration.name = data.get('name', registration.name)
            registration.email = data.get('email', registration.email)
            registration.date_of_birth = data.get('date_of_birth', registration.date_of_birth)
            registration.phone_number = data.get('phone_number', registration.phone_number)
            registration.address = data.get('address', registration.address)
            registration.is_active = data.get('is_active', registration.is_active)

            db.session.commit()
            return jsonify({'message': 'Registration updated successfully'}), 200
        except Exception as e:
            db.session.rollback()
            return jsonify({'error': str(e)}), 400
    return jsonify({'error': 'Registration not found'}), 404

# Delete
@app.route('/registrations/<int:id>', methods=['DELETE'])
def delete_registration(id):
    registration = Registration.query.get(id)
    if registration:
        db.session.delete(registration)
        db.session.commit()
        return jsonify({'message': 'Registration deleted successfully'}), 204
    return jsonify({'error': 'Registration not found'}), 404

if __name__ == '__main__':
    app.run(debug=True)
