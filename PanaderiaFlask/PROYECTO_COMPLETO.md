# PROYECTO PANADERÍA - CÓDIGO COMPLETO

## Estructura de archivos que necesitas crear:

```
panaderia/
├── app.py
├── main.py
├── models.py
├── forms.py
├── requirements.txt
├── auth/
│   ├── __init__.py
│   └── routes.py
├── products/
│   ├── __init__.py
│   └── routes.py
├── cart/
│   ├── __init__.py
│   └── routes.py
├── admin/
│   ├── __init__.py
│   └── routes.py
├── templates/
│   ├── base.html
│   ├── index.html
│   ├── auth/
│   │   ├── login.html
│   │   └── register.html
│   ├── products/
│   │   ├── index.html
│   │   └── category.html
│   ├── cart/
│   │   └── index.html
│   ├── orders/
│   │   ├── confirmation.html
│   │   └── history.html
│   └── admin/
│       ├── dashboard.html
│       ├── products.html
│       ├── categories.html
│       ├── product_form.html
│       └── category_form.html
└── static/
    ├── css/
    │   └── custom.css
    └── js/
        └── cart.js
```

---

## requirements.txt
```
flask
flask-sqlalchemy
flask-login
flask-migrate
flask-wtf
gunicorn
psycopg2-binary
sqlalchemy
werkzeug
wtforms
email-validator
```

---

## app.py
```python
import os
import logging
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
from flask_login import LoginManager
from flask_migrate import Migrate
from sqlalchemy.orm import DeclarativeBase
from werkzeug.middleware.proxy_fix import ProxyFix

# Configure logging
logging.basicConfig(level=logging.DEBUG)

class Base(DeclarativeBase):
    pass

db = SQLAlchemy(model_class=Base)
login_manager = LoginManager()
migrate = Migrate()

def create_app():
    app = Flask(__name__)
    app.secret_key = os.environ.get("SESSION_SECRET", "dev-secret-key")
    app.wsgi_app = ProxyFix(app.wsgi_app, x_proto=1, x_host=1)
    
    # Database configuration
    app.config["SQLALCHEMY_DATABASE_URI"] = os.environ.get("DATABASE_URL", "sqlite:///bakery.db")
    app.config["SQLALCHEMY_ENGINE_OPTIONS"] = {
        "pool_recycle": 300,
        "pool_pre_ping": True,
    }
    app.config["SQLALCHEMY_TRACK_MODIFICATIONS"] = False
    
    # Initialize extensions
    db.init_app(app)
    login_manager.init_app(app)
    migrate.init_app(app, db)
    
    # Login manager configuration
    login_manager.login_view = 'auth.login'
    login_manager.login_message = 'Please log in to access this page.'
    login_manager.login_message_category = 'info'
    
    @login_manager.user_loader
    def load_user(user_id):
        from models import User
        return User.query.get(int(user_id))
    
    # Register blueprints
    from auth import bp as auth_bp
    app.register_blueprint(auth_bp, url_prefix='/auth')
    
    from products import bp as products_bp
    app.register_blueprint(products_bp, url_prefix='/products')
    
    from cart import bp as cart_bp
    app.register_blueprint(cart_bp, url_prefix='/cart')
    
    from admin import bp as admin_bp
    app.register_blueprint(admin_bp, url_prefix='/admin')
    
    # Main routes
    @app.route('/')
    def index():
        from flask import render_template
        from models import Product, Category
        
        featured_products = Product.query.filter_by(featured=True).limit(6).all()
        categories = Category.query.all()
        
        return render_template('index.html', 
                             featured_products=featured_products,
                             categories=categories)
    
    @app.route('/orders')
    def order_history():
        from flask import render_template
        from flask_login import login_required, current_user
        from models import Order
        
        if not current_user.is_authenticated:
            return login_manager.unauthorized()
            
        orders = Order.query.filter_by(user_id=current_user.id).order_by(Order.created_at.desc()).all()
        return render_template('orders/history.html', orders=orders)
    
    with app.app_context():
        import models
        db.create_all()
        
        # Create default admin user and sample data
        from werkzeug.security import generate_password_hash
        from models import User, Category, Product
        
        if not User.query.filter_by(email='admin@bakery.com').first():
            admin_user = User(
                username='admin',
                email='admin@bakery.com',
                password_hash=generate_password_hash('admin123'),
                is_admin=True
            )
            db.session.add(admin_user)
            
        # Create default categories
        if not Category.query.first():
            categories = [
                Category(name='Bread', description='Fresh baked bread'),
                Category(name='Cakes', description='Delicious cakes for all occasions'),
                Category(name='Pastries', description='Sweet and savory pastries'),
                Category(name='Cookies', description='Homemade cookies')
            ]
            for category in categories:
                db.session.add(category)
            
            db.session.commit()
            
            # Add sample products
            sample_products = [
                Product(name='Sourdough Bread', description='Traditional artisan sourdough with a crispy crust', 
                       price=5.99, category_id=1, featured=True, active=True),
                Product(name='Chocolate Croissant', description='Buttery pastry filled with rich dark chocolate', 
                       price=3.50, category_id=3, featured=True, active=True),
                Product(name='Vanilla Birthday Cake', description='Classic vanilla cake with buttercream frosting', 
                       price=25.00, category_id=2, featured=True, active=True),
                Product(name='Chocolate Chip Cookies', description='Fresh baked cookies with premium chocolate chips', 
                       price=2.99, category_id=4, featured=False, active=True),
                Product(name='Whole Wheat Bread', description='Healthy whole wheat bread with seeds', 
                       price=4.99, category_id=1, featured=False, active=True),
                Product(name='Strawberry Tart', description='Fresh strawberries on vanilla custard', 
                       price=4.75, category_id=3, featured=True, active=True),
                Product(name='Red Velvet Cake', description='Rich red velvet cake with cream cheese frosting', 
                       price=28.00, category_id=2, featured=False, active=True),
                Product(name='Oatmeal Cookies', description='Healthy oatmeal cookies with raisins', 
                       price=2.75, category_id=4, featured=False, active=True)
            ]
            
            for product in sample_products:
                db.session.add(product)
                
        db.session.commit()
    
    return app

# Create app instance
app = create_app()
```

---

## main.py
```python
from app import app

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000, debug=True)
```

---

## models.py
```python
from app import db
from flask_login import UserMixin
from datetime import datetime
from werkzeug.security import generate_password_hash, check_password_hash

class User(UserMixin, db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(64), unique=True, nullable=False)
    email = db.Column(db.String(120), unique=True, nullable=False)
    password_hash = db.Column(db.String(256), nullable=False)
    is_admin = db.Column(db.Boolean, default=False)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    
    # Relationships
    orders = db.relationship('Order', backref='user', lazy=True)
    cart_items = db.relationship('CartItem', backref='user', lazy=True, cascade='all, delete-orphan')
    
    def set_password(self, password):
        self.password_hash = generate_password_hash(password)
    
    def check_password(self, password):
        return check_password_hash(self.password_hash, password)
    
    def __repr__(self):
        return f'<User {self.username}>'

class Category(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(80), nullable=False, unique=True)
    description = db.Column(db.Text)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    
    # Relationships
    products = db.relationship('Product', backref='category', lazy=True)
    
    def __repr__(self):
        return f'<Category {self.name}>'

class Product(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(100), nullable=False)
    description = db.Column(db.Text)
    price = db.Column(db.Numeric(10, 2), nullable=False)
    image_url = db.Column(db.String(200))
    featured = db.Column(db.Boolean, default=False)
    active = db.Column(db.Boolean, default=True)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    
    # Foreign Keys
    category_id = db.Column(db.Integer, db.ForeignKey('category.id'), nullable=False)
    
    # Relationships
    cart_items = db.relationship('CartItem', backref='product', lazy=True)
    order_items = db.relationship('OrderItem', backref='product', lazy=True)
    
    def __repr__(self):
        return f'<Product {self.name}>'

class CartItem(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    quantity = db.Column(db.Integer, nullable=False, default=1)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    
    # Foreign Keys
    user_id = db.Column(db.Integer, db.ForeignKey('user.id'), nullable=False)
    product_id = db.Column(db.Integer, db.ForeignKey('product.id'), nullable=False)
    
    @property
    def total_price(self):
        return self.quantity * self.product.price
    
    def __repr__(self):
        return f'<CartItem {self.product.name} x{self.quantity}>'

class Order(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    total_amount = db.Column(db.Numeric(10, 2), nullable=False)
    status = db.Column(db.String(20), default='pending')  # pending, confirmed, preparing, ready, delivered
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    
    # Foreign Keys
    user_id = db.Column(db.Integer, db.ForeignKey('user.id'), nullable=False)
    
    # Relationships
    order_items = db.relationship('OrderItem', backref='order', lazy=True, cascade='all, delete-orphan')
    
    def __repr__(self):
        return f'<Order {self.id} - ${self.total_amount}>'

class OrderItem(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    quantity = db.Column(db.Integer, nullable=False)
    price = db.Column(db.Numeric(10, 2), nullable=False)  # Price at time of order
    
    # Foreign Keys
    order_id = db.Column(db.Integer, db.ForeignKey('order.id'), nullable=False)
    product_id = db.Column(db.Integer, db.ForeignKey('product.id'), nullable=False)
    
    @property
    def total_price(self):
        return self.quantity * self.price
    
    def __repr__(self):
        return f'<OrderItem {self.product.name} x{self.quantity}>'
```

---

## forms.py
```python
from flask_wtf import FlaskForm
from wtforms import StringField, PasswordField, TextAreaField, DecimalField, SelectField, IntegerField, BooleanField
from wtforms.validators import DataRequired, Email, Length, EqualTo, NumberRange, ValidationError
from models import User, Category

class LoginForm(FlaskForm):
    email = StringField('Email', validators=[DataRequired(), Email()])
    password = PasswordField('Password', validators=[DataRequired()])

class RegistrationForm(FlaskForm):
    username = StringField('Username', validators=[DataRequired(), Length(min=4, max=20)])
    email = StringField('Email', validators=[DataRequired(), Email()])
    password = PasswordField('Password', validators=[DataRequired(), Length(min=6)])
    password2 = PasswordField('Repeat Password', validators=[DataRequired(), EqualTo('password')])
    
    def validate_username(self, username):
        user = User.query.filter_by(username=username.data).first()
        if user:
            raise ValidationError('Username already exists. Please choose a different one.')
    
    def validate_email(self, email):
        user = User.query.filter_by(email=email.data).first()
        if user:
            raise ValidationError('Email already registered. Please choose a different one.')

class ProductForm(FlaskForm):
    name = StringField('Product Name', validators=[DataRequired(), Length(max=100)])
    description = TextAreaField('Description')
    price = DecimalField('Price', validators=[DataRequired(), NumberRange(min=0.01)])
    image_url = StringField('Image URL', validators=[Length(max=200)])
    category_id = SelectField('Category', coerce=int, validators=[DataRequired()])
    featured = BooleanField('Featured Product')
    active = BooleanField('Active', default=True)
    
    def __init__(self, *args, **kwargs):
        super(ProductForm, self).__init__(*args, **kwargs)
        self.category_id.choices = [(c.id, c.name) for c in Category.query.all()]

class CategoryForm(FlaskForm):
    name = StringField('Category Name', validators=[DataRequired(), Length(max=80)])
    description = TextAreaField('Description')

class CartItemForm(FlaskForm):
    quantity = IntegerField('Quantity', validators=[DataRequired(), NumberRange(min=1)])
```

**CONTINÚA EN LA SIGUIENTE PARTE...**