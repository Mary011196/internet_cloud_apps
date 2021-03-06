% Διαδικτυακές και Νεφοϋπολογιστικές Εφαρμογές
% Αν. Καθηγητής Π. Λουρίδας
% Οικονομικό Πανεπιστήμιο Αθηνών

# Object Relational Mapping

## Γενικά

* Οι περισσότερες βάσεις δεδομένουν χρησιμοποιούν το *σχεσιακό
  μοντέλο* (relational model).

* Στα προγράμματα που γράφουμε όμως, δεν έχουμε σχέσεις. Έχουμε
  αντικείμενα και άλλες δομές δεδομένων.

* Πρέπει λοιπόν να μεταφράζουμε μεταξύ των σχέσεων και των
  αντικειμένων του προγράμματός μας.


## Επιπλέον προβλήματα

* Στις σχεσιακές βάσεις δεδομένων χρησιμοποιούμε SQL.

* Στα προγράμματά μας χρησιμοποιούμε άλλες γλώσσες.

* Πρέπει λοιπόν ταυτόχρονα να χρησιμοποιούμε SQL συν ό,τι άλλες
  γλώσσες χρειαζόμαστε.

* Ο κώδικας SQL ελέγχεται μόνο κατά τη διάρκεια εκτέλεσης του
  προγράμματος (runtime).


## Απεικόνιση αντικειμένων-οντοτήτων

* Η *απεικόνιση αντικεμένων-οντοτήτων* (Object Relational Mapping,
  ORM) είναι μια σειρά τεχνολογιών με τις οποίες τα αντικείμενα που
  χρησιμοποιούμε στην εφαρμογή μας μεταφράζονται αυτομάτως σε σχέσεις.

* Υπάρχουν πολλές υλοποιήσεις, σε πολλές γλώσσες.


## SQLAlchemy

* Η πιο δημοφιλής βιβλιοθήκη για ORM στην Python είναι η
  [SQLAlchemy](http://www.sqlalchemy.org/).

* Αν θέλουμε να τη χρησιμοποιήσουμε με το Flask, μπορούμε να
  εγκαταστήσουμε το πακέτο
  [Flask-SQLAlchemy](http://flask-sqlalchemy.pocoo.org/).

    ```bash
    pip install flask-sqlalchemy
    ```
	
## Εγκατάσταση IPython

* Θα χρειαστεί να χρησιμοποιήσουμε αρκετές φορές τη γραμμή εντολών της
  Python.

* Επειδή η γραμμή εντολών της Python είναι λίγο δύσχρηστη, στην πράξη
  χρησιμοποιούμε το εργαλείο [IPython](https://ipython.org/).

* Για να το εγκαταστήσουμε δίνουμε:

    ```bash
    pip install ipython
    ```
    
* Στο εξής μπορούμε να δίνουμε `ipython` για να ξεκινάμε μία γραμμή
  εντολών της Python.

<div class="notes">

Το IPython έχει πολλές δυνατότητες που κάνουν την αλληλεπίδραση με την
Python ευκολότερη. Επιγραμματικά:
	
* Συμπλήρωση εντολών (command line completion). Καθώς πληκτρολογούμε
κάτι σε Python, μπορούμε να πατάμε Tab οπότε θα φαίνονται η διαθέσιμες
επιλογές για τη συμπλήρωση της εντολής μας.
	  
* Πλήρες ιστορικό εντολών (command history). Μπορούμε να πλοηγηθούμε
στις εντολές που έχουμε δώσει χρησιμοποιώντας τα βελάκια άνω και κάτω.
	  
* Αντιγραφή και επικόλληση (copy and paste). Μπορούμε να αντιγράφουμε
εύκολα κώδικα από και προς το IPython.
	  
Για μία σύντομη εισαγωγή, μπορείτε να δείτε
την
[online τεκμηρίωση](http://ipython.readthedocs.io/en/stable/interactive/tutorial.html).

</div>


# Microblog με Flask-SQLAlchemy

## Γενικά

* Θα μετατρέψουμε την εφαρμογή που γράψαμε στην προηγούμενη διάλεξη
  ώστε να χρησιμοποιεί SQLAlchemy.
  
* Θα δούμε ότι με τον τρόπο αυτό δεν θα γράψουμε καθόλου κώδικα SQL.

## Δημιουργία περιβάλλοντος

* Δημιουργείστε την παρακάτω δομή καταλόγων.

* Στους καταλόγους `static`
  και `templates` θα αντιγράψετε τα αρχεία που δημιουργήσατε στην
  έκδοση του microblog που είδαμε στην πρώτη διάλεξη.

    ```
    /flaskr_orm_1
        /static
        /templates
    ```

## Προοίμιο εφαρμογής

* Στον κατάλογο `flaskr_orm_1` δημιουργείστε ένα αρχείο `flaskr.py`, στο
  οποίο θα γράφετε αυτά που ακολουθούν.

    ```python
    from flask import Flask
    from flask import Flask, request, session, g, redirect, url_for, abort, \
         render_template, flash

    from flask_sqlalchemy import SQLAlchemy

    app = Flask(__name__)
    app.config.from_object(__name__)

    app.config.update(dict(
        SECRET_KEY='development key',
        USERNAME='admin',
        PASSWORD='default',
        SQLALCHEMY_DATABASE_URI = 'sqlite:///flaskr.db',
        SQLALCHEMY_TRACK_MODIFICATIONS = False
    ))

    app.config.from_envvar('FLASKR_SETTINGS', silent=True)

    db = SQLAlchemy(app)
    ```

<div class="notes">

Το προοίμιο είναι το ίδιο, με μόνες διαφορές τις εξής:

* Χρησιμοποιούμε και τη βιβλιοθήκη SQLAlchemy, οπότε απαιτείται το
  κατάλληλο `import`.
  
* Οι ρυθμίσεις για τη βάση αλλάζουν. Το `SQLALCHEMY_DATABASE_URI`
  δείχνει την τοποθεσία της βάσης. Στη συγκεκριμένη περίπτωση, θα
  είναι το αρχείο `flaskr.db` στον κατάλογο της εφαρμογής μας.
  
* Η ρύθμιση για το `SQLALCHEMY_TRACK_MODIFICATIONS` είναι μια τεχνική
  λεπτομέρεια. Αφορά ένα λεπτό χαρακτηριστικό του SQLAlchemy, δεν
  χρειάζεται να σας απασχολήσει.

* Η γραμμή `db = SQLAlchemy(app)` δημιουργεί το αντικείμενο `db` με το
  οποίο θα αλληλεπιδρούμε με τη βάση δεδομένων μας.

</div>

## Μοντέλο βάσης

* Αυτή τη φορά δεν θα χρειαστεί να γράψουμε κώδικα SQL για τον ορισμό
  του πίνακα `entries`.

* Ο πίνακας θα οριστεί μέσω αντίστοιχης κλάσης Python.

    ```python
    class Entry(db.Model):
        __tablename__ = 'entries'
        id = db.Column(db.Integer, primary_key=True)
        title = db.Column(db.String(120), nullable=False)
        text = db.Column(db.Text(), nullable=False)

        def __init__(self, title="", text=""):
            self.title = title
            self.text = text

        def __repr__(self):
            return '<Entry %r %r>' % (self.title, self.text)
    ```

<div class="notes">

Στο SQLAlchemy θα απεικονίζουμε τους πίνακες της βάσης μέσω κλάσεων
στην Python. Οι κλάσεις θα πρέπει να είναι απόγονοι της κλάσης
`db.Model`. Η αντιστοίχηση μεταξύ της κλάσης και του πίνακα στη βάση
δεδομένων γίνεται με την ιδιότητα `__tablename__`. Προσέξτε τη σύμβαση
στην ονοματολογία: τα ονόματα των κλάσεων στις γλώσσες
προγραμμματισμού είναι στον *ενικό*. Τα ονόματα των πινάκων είναι
συνήθως στον *πληθυντικό*.

Στη συνέχεια, ορίζουμε της στήλες του πίνακα μέσω ιδιοτήτων. Ο πίνακας
`entries` θέλουμε να έχει ως στήλες έναν κωδικό (`id`), έναν τίτλο
(`title`), και ένα κείμενο (`text`). Το κάθε ένα από αυτά ορίζεται
μέσω κατάλληλης κλήσης της μεθόδου `db.Column()`. Η πρώτη παράμετρος
της μεθόδου δίνει τον τύπο της στήλης και επιπλέον παράμετροι δίνουν
ιδιότητες, όπως αν πρόκειται για κλειδί, αν επιτρέπονται οι τιμές
null, κ.λπ. Περισσότερες πληροφορίες μπορείτε να βρείτε
στην
[τεκμηρίωση του Flask-SQLAlchemy](http://flask-sqlalchemy.pocoo.org/2.1/models/#simple-example).
Αν θέλετε να εμβαθύνετε, δείτε
την
[τεκμηρίωση του SQLAlchemy](http://docs.sqlalchemy.org/en/latest/index.html).

Η κλάση που ορίζουμε θα πρέπει να έχει έναν κατασκευαστή
(constructor), ο οποίος όπως πάντα στην Python ορίζεται με τη μέθοδο
`__init()__`. Ο κατασκευαστής αυτός συνήθως δεν κάνει τίποτε άλλο από
αρχικοποίηση των πεδίων με τις παραμέτρους που του περνάμε. Το πεδίο
`id` συνήθως δεν χρειάζεται αρχικοποίηση, καθώς το χειρίζεται
αυτομάτως η βάση.

Η μέθοδος `__repr__()` θα μας εκτυπώνει μια αναπάρασταση του
κάθε αντικειμένου τύπου `Entry`. Μοιάζει με τη μέθοδο `__str__`,
αλλά ενώ η μέθοδος `__str__` δημιουργεί απλώς μια συμβολοσειρά από το
αντικείμενό μας, η μέθοδος `__repr__` δημιουργεί μια συμβολοσειρά η
οποία περιέχει όλη την πληροφορία που χρειαζόμαστε για να το
κατασκευάσουμε, αν θέλουμε. Αντίστοιχα με το `%s`, το οποίο όταν
εμφανίζεται σε μία συμβολοσειρά θα αντικασταθεί με μία κλήση της
`__str__`, το `%r` θα αντικατασταθεί με μία κλήση της `__repr__`.

</div>


## Δημιουργία βάσης

* Έχοντας γράψει τον παραπάνω κώδικα, μπορούμε να δημιουργήσουμε τη
  βάση μας κατ' ευθείαν από μία γραμμή εντολών Python.

* Ξεκινάμε την Python μέσα στον κατάλογο `flaskr_orm_1`
  πληκτρολογόντας `python`, ή καλύτερα `ipython`.

* Για να είναι εμφανές ότι μιλάμε για τη γραμμή εντολών της Python,
  η είσοδός μας και η έξοδός μας εκεί θα ξεκινούν με `>>>` σε ό,τι
  ακολουθεί. 

* Στη συνέχεια γράφουμε:

    ```python
    >>> from flaskr import db
    >>> db.create_all()
    ```

## Εισαγωγή δεδομένων

* Τώρα μπορούμε να χειριστούμε πλήρως τη βάση μας από τη γραμμή
  εντολών της Python.

* Για παράδειγμα, για να εισάγουμε δύο αναρτήσεις, γράφουμε:

    ```python
    >>> from flaskr import Entry

    >>> entry_1 = Entry("First Entry", "This is the first entry")
    >>> entry_2 = Entry("Second Entry", "This is the second entry")

    >>> db.session.add(entry_1)
    >>> db.session.add(entry_2)
    >>> db.session.commit()
    ```

## Αναζήτηση δεδομένων

* Για να κάνουμε μια αναζήτηση στα δεδομένα, γράφουμε:

    ```python
    >>> entries = Entry.query.all()
    ```
	
<div class="notes">

Στο συγκεκριμένο παράδειγμα αναζητούμε το σύνολο των δεδομένων. Φυσικά
μπορούμε να κάνουμε πιο λεπτομερείες και περίπλοκες αναζητήσεις. Δείτε
τη
[σχετική τεκμηρίωση του Flask-SQLAlchemy](http://flask-sqlalchemy.pocoo.org/2.1/queries/#querying-records) και
την [πλήρη τεκμηρίωση του SQLAlchemy](http://docs.sqlalchemy.org/en/latest/orm/tutorial.html#common-filter-operators).

</div>
	
* Πράγματι, ενόσω είμαστε στη γραμμή εντολών μπορούμε να
  επιβεβαιώσουμε τις εγγραφές:

    ```python
    >>> entries
    ```

## Διαγραφή δεδομένων

* Για να διαγράψουμε τα δεδομένα, γράφουμε:

    ```python
    >>> Entry.query.delete()
    >>> db.session.commit()
    ```

<div class="notes">

* Πάλι σβήνουμε το σύνολο των εγγραφών στον πίνακα `entries`. Αν
  θέλαμε να σβήσουμε μία συγκεκριμένη εγγραφή, έστω την `entry_1`, θα
  δίναμε:

    ```python
    >>> print("Hello")
    >>> db.session.delete(entry_1)
    ```
	
</div>

* Και πάλι επιβεβαιώνουμε:

    ```python
    >>> entries = Entry.query.all()
    >>> entries
    ```


## Εμφάνιση αναρτήσεων

* Θα συνεχίσουμε με τη συγγραφή των μεθόδων για το χειρισμό των
  λειτουργιών της υπηρεσίας.

* Η εμφάνιση των αναρτήσεων, με τη χρήση πλέον Flask-SQLAlchemy,
  είναι:

    ```python
    @app.route('/')
    def show_entries():
        entries = Entry.query.all()
        return render_template('show_entries.html', entries=entries)
    ```

<div class="notes">

Η μέθοδος είναι στην ουσία η ίδια με αυτήν που είχαμε στην προηγούμενη
έκδοση της εφαρμογής, αλλά αντί να γράψουμε την εντολή της αναζήτησης
σε SQL γράφουμε μόνο τις αντίστοιχες εντολές σε Python.

</div>

## Προσθήκη ανάρτησης

* Αντίστοιχα, η προσθήκη αναρτήσεων γίνεται:

    ```python
    @app.route('/add', methods=['POST'])
    def add_entry():
        if not session.get('logged_in'):
            abort(401)
        entry = Entry(request.form['title'], request.form['text'])
        db.session.add(entry)
        db.session.commit()
        flash('New entry was successfully posted')
        return redirect(url_for('show_entries'))
    ```

<div class="notes">

Και πάλι, οι αλλαγές σε σχέση με την προηγούμενη εκδοσή αφορούν στη
χρήση Python αντί για τη χρήση SQL. Δημιουργούμε ένα αντικείμενο της
κλάσης `Entry` χρησιμοποιώντας τα στοιχεία που έχει συμπληρώσει ο
χρήστης στη φόρμα της εφαρμογής, και στη συνέχεια εισάγουμε τη νέα
ανάρτηση στη βάση και επικυρώνουμε την εισαγωγή.

</div>


## Λοιπές λειτουργίες και εκκίνηση

* Οι άλλες δύο λειτουργίες, είσοδος και έξοδος, παραμένουν ως έχουν.

* Συνεπώς, απλώς αντιγράψτε τες από την προηγούμενη έκδοση της
  εφαρμογής.

* Για να τρέξετε την εφαρμογή δίνετε ότι και στην προηγούμενη έκδοση.

<div class="notes">

Υπενθυμίζουμε:

*  Για να ξεκινήσουμε την εφαρμογή σε MS-Windows πρέπει καταρχήν να δώσουμε
  από τη γραμμή εντολών:

    ```
    set FLASK_APP=flaskr.py
    set FLASK_DEBUG=1
    ```

* Στη συνέχεια για να ξεκινήσουμε την εφαρμογή θα πρέπει να βρισκόμαστε μέσα
  στον κατάλογο `flaskr` και να δώσουμε:

    ```bash
    python -m flask run
    ```

* Για να ξεκινήσουμε την εφαρμογή σε Mac και Linux πρέπει καταρχήν να δώσουμε
  από τη γραμμή εντολών:

    ```bash
    export FLASK_APP=flaskr.py
    export FLASK_DEBUG=1
    ```

* Στη συνέχεια για να ξεκινήσουμε την εφαρμογή θα πρέπει να βρισκόμαστε μέσα
  στον κατάλογο `flaskr` και να δώσουμε:

    ```bash
    flask run
    ```
	 
</div>

# Microblog με Πολλαπλούς Χρήστες


## Γενικά

* Η εφαρμογή μας, μέχρι τώρα, μπορεί να λειτουργήσει μόνο ως προσωπικό
  blog, αφού έχει μόνο έναν χρήστη.
  
* Τώρα θα την επεκτείνουμε ώστε να μπορούμε να έχουμε πολλούς χρήστες,
  οι οποίοι μπορούν να δημιουργούν αναρτήσεις.
  
* Συνεπώς δεν θα είναι πλέον απλώς ένα blog, αλλά περισσότερο ένας
  διαμοιραζόμενος πίνακας ανακοινώσεων (community board).


## Δημιουργία περιβάλλοντος

* Δημιουργείστε την παρακάτω δομή καταλόγων.

    ```
    /flaskr_orm_2
        /static
        /templates
    ```

## Προοίμιο εφαρμογής

* Στον κατάλογο `flaskr_orm_2` δημιουργείστε ένα αρχείο `flaskr.py`, στο
  οποίο θα γράφετε αυτά που ακολουθούν.

    ```python

    from flask import Flask
    from flask import Flask, request, session, g, redirect, url_for, abort, \
         render_template, flash

    from flask_sqlalchemy import SQLAlchemy

    from datetime import datetime

    app = Flask(__name__)
    app.config.from_object(__name__)

    app.config.update(dict(
        SECRET_KEY='development key',
        SQLALCHEMY_DATABASE_URI = 'sqlite:///flaskr.db',
        SQLALCHEMY_TRACK_MODIFICATIONS = False
    ))

    app.config.from_envvar('FLASKR_SETTINGS', silent=True)

    db = SQLAlchemy(app)
    ```

<div class="notes">

Αφού θα έχουμε πολλούς χρήστες, δεν μπορούμε πλέον να αποθηκεύουμε τα
στοιχεία τους στις ρυθμίσεις της εφαρμογής (που ούτως ή άλλως δεν
είναι καλή ιδέα). Συνεπώς, αφαιρέσαμε τις σχετικές γραμμές. Επίσης θα
χρησιμοποιήσουμε τη βιβλιοθήκη `datetime` για να αποθηκεύουμε τη
χρονική στιγμή δημιουργίας κάθε ανάρτησης.

</div>


## Μοντέλο βάσης: αναρτήσεις

* Τώρα θα επεκτείνουμε το μοντέλο της βάσης μας ώστε να έχουμε πολλούς
  χρήστες.

* Ο χρήστης θα μπορεί να δημιουργεί αναρτήσεις.

* Κάθε ανάρτηση θα ανήκει σε έναν χρήστη.

* Το μοντέλο των αναρτήσεων θα είναι το ακόλουθο.


## Κώδικας αναρτήσεων

```python
class Entry(db.Model):
    __tablename__ = 'entries'
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(120), nullable=False)
    text = db.Column(db.Text(), nullable=False)
    datetime = db.Column(db.DateTime(), nullable=False)

    user_id = db.Column(db.Integer, db.ForeignKey('users.id'))

    def __init__(self, title, text, user_id, datetime):
        self.title = title
        self.text = text
        self.user_id = user_id
        self.datetime = datetime

    def __repr__(self):
        return '<Entry %r %r %r %r>' % (self.title, self.text,
                                        self.user_id, self.datetime)
```

<div class="notes">

Εμπλουτίζομουμε την κλάση `Entry` με δύο επιπλέον πεδία:

* `datetime`, όπου θα αποθηκεύεται η χρονική στιγμή δημιουργίας της
  ανάρτησης.
  
* `user_id`, όπου θα αποθηκεύεται ο κωδικός στη βάση του χρήστη που
  έκανε την ανάρτηση. Αυτό θα είναι ένα ξένο κλειδί (foreign key) στον
  πίνακα `users`, όπου θα αποθηκεύουμε τους χρήστες, όπως θα δούμε στη
  συνέχεια. 

</div>


## Μοντέλο βάσης: χρήστες

* Ο κάθε χρήστης θα έχει όνομα, επώνυμο, κωδικό, e-mail.

* Επίσης κάθε χρήστης έχει ένα σύνολο αναρτήσεων που έχει δημιουργήσει.

## Κώδικας χρηστών

```python
class User(db.Model):
    __tablename__ = 'users'
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(20),
                         nullable=False,
                         unique=True,
                         index=True)
    password = db.Column(db.String(30), nullable=False)
    name = db.Column(db.String(20), nullable=False)
    surname = db.Column(db.String(30), nullable=False)
    email = db.Column(db.String(30), nullable=False)
    entries = db.relationship('Entry', backref='user',
                              lazy='dynamic')

    def __init__(self, username, name, surname, email, password):
        self.username = username
        self.name = name
        self.surname = surname
        self.email = email
        self.password = password

    def __repr__(self):
        return '<User %r %r %r %r>' % (self.username, self.email,
                                       self.name, self.surname)
```

<div class="notes">

Ορίζουμε τα πεδία του χρήστη με τους περιορισμούς που τους αρμόζουν:
για παράδειγμα, το `username` θέλουμε να μην είναι null, να είναι
μοναδικό, και να δημιουργηθεί ένα ευρετήριο (index) επάνω του.

Το πεδίο `entries` είναι αυτό που θα περιέχει το σύνολο των αναρτήσεων
που έχει κάνει ένας χρήστης. Ορίζουμε λοιπόν ότι αντιστοιχεί σε μία
σχέση, και όχι σε μία στήλη. Στη σχέση δίνουμε το όνομα της κλάσης
στην οποία θα δείχνει. Επιπλέον, με την παράμετρο `backref='user'`
ορίζουμε ότι στην κλάση `Entry` θα δημιουργηθεί δυναμικά ένα πεδίο
`user` το οποίο θα δείχνει στον χρήστη που έκανε τη συγκεκριμένη
ανάρτηση. Η παράμετρος `lazy='dynamic'` ορίζει ότι οι αναρτήσεις για
έναν χρήστη θα διαβάζονται από τη βάση μόνο όταν ζητηθούν, και όχι
απλώς όταν διαβάζουμε έναν χρήστη. Ο τρόπος με τον οποίο διαβάζονται
από τη βάση τα αντικείμενα μιας σχέσης μπορεί να είναι σημαντικός
παράγοντας βελτιστοποίησης. Για περισσότερες πληροφορίες, βλ. τη
σχετική τεκμηρίωση του
[lask-SQLAlchemy](http://flask-sqlalchemy.pocoo.org/2.1/models/#one-to-many-relationships) και
του [SQLAlchemy](http://docs.sqlalchemy.org/en/latest/orm/tutorial.html#building-a-relationship).

</div>


## Δημιουργία βάσης

* Για να δημιουργήσουμε τη βάση θα χρησιμοποιήσουμε και πάλι τη γραμμή
  εντολών της Python ή IPython:

    ```python
    >>> from flaskr import *
    >>> db.create_all()
    ```

## Εισαγωγή χρηστών

* Αυτή τη στιγμή δεν θα υλοποιήσουμε τη διαδικασία εγγραφής χρηστών
  μέσα στην εφαρμογή, οπότε απλώς θα τους εισάγουμε απ' ευθείας μέσω
  της γραμμής εντολών Python:
  

    ```python

    >>> user_1 = User("louridas", "Panos", "Louridas", "louridas@aueb.gr", "12345")

    >>> user_2 = User("johndoe", "John", "Doe", "johndoe@gmail.com", "54321")

    >>> db.session.add(user_1)
    >>> db.session.add(user_2)
    >>> db.session.commit()
    ```

<div class="notes">

Προφανώς τέτοιοι κωδικοί είναι εντελώς απαράδεκτοι.

</div>


## Εμφάνιση αναρτήσεων

* Για να δούμε τις αναρτήσεις θα κάνουμε αναζήτηση στη βάση μέσω
  SQLAlchemy:
  

    ```python
    @app.route('/')
    def show_entries():
        entries = Entry.query.order_by(Entry.datetime.desc()).all()
        return render_template('show_entries.html', entries=entries)
    ```

<div class="notes">

Εδώ βλέπετε ένα επιπλέον χαρακτηριστικό των αναζητήσεων: μπορείτε να
ορίσετε διαφόρους τρόπους ταξινόμησης των αποτελεσμάτων. Όπως συνήθως,
περισσότερες πληφορορίες θα βρείτε στην τεκμηρίωση
του
[Flask-SQLAlchemy](http://flask-sqlalchemy.pocoo.org/2.1/queries/#querying-records) και
του
[SQLAlchemy](http://docs.sqlalchemy.org/en/latest/orm/tutorial.html#querying). 

</div>

## Προσθήκη ανάρτησης

* Όταν ο χρήστης δημιουργεί μια νέα ανάρτηση θα πρέπει να αποθηκεύουμε
  στη βάση:
      * τον τίτλο
      * το κείμενό της
      * ποιος χρήστης έκανε την ανάρτηση
      * ποια ακριβώς χρονική στιγμή

* Ταυτόχρονα, η εισαγωγή θα γίνει μέσω SQLAlchemy.


## Κώδικας προσθήκης

```python
@app.route('/add', methods=['POST'])
def add_entry():
    user_id = session.get('user_id')
    if user_id is None:
        abort(401)
    entry = Entry(request.form['title'],
                  request.form['text'],
                  user_id,
                  datetime.now()
    )
    db.session.add(entry)
    db.session.commit()
    flash('New entry was successfully posted')
    return redirect(url_for('show_entries'))
```

<div class="notes">

Δημιουργούμε ένα αντικείμενο της κλάσης `Entry` με τα δεδομένα που
έχει εισάγει ο χρήστης από τη σχετική φόρμα. Η κλήση `datetime.now()`
δίνει την τρέχουσα χρονική στιγμή.

Προσέξτε τον έλεγχο για το αν ο χρήστης έχει περάσει τη διαδικασία
εισόδου στο σύστημα. Στις προηγούμενες εκδόσεις της εφαρμογής ελέγχαμε
απλώς αν το αντικείμενο `session` είχε ένα κλειδί `logged_in`. Τώρα
ελέγχουμε αν υπάρχει μέσα σε αυτό το `user_id`. Θα δούμε παρακάτω ότι
το εισάγουμε κατά τη διαδικασία εισόδου στο σύστημα.

</div>


## Είσοδος

```python
@app.route('/login', methods=['GET', 'POST'])
def login():
    error = None
    if request.method == 'POST':
        user = User.query.filter_by(username=request.form['username']).first()
        if user is None:
            error = 'Invalid username'
        elif request.form['password'] != user.password:
            error = 'Invalid password'
        else:
            session['user_id'] = user.id
            session['username'] = user.username
            flash('You were logged in')
            return redirect(url_for('show_entries'))
    return render_template('login.html', error=error)
```

<div class="notes">

Για να διαπιστώσουμε αν ένας χρήστης έχει δικαίωμα εισόδου στην
εφαρμογή, πραγματοποιούμε αναζήτηση στη βάση με βάση το `username` που
μας έχει δώσει. Οι αναζητήσεις στη βάση επιστρέφουν εν δυνάμει πολλά
αποτελέσματα, οπότε εμείς ζητούμε απλώς το πρώτο (και μοναδικό). Αν
δεν υπάρχει, επιστρέφουμε μήνυμα λάθους. Διαφορετικά, αν υπάρχει,
ελέγχουμε αν ο κωδικός που έχει δώσει ο χρήστης συμφωνεί με τον κωδικό
που έχει αποθηκευτεί στη βάση, και οποίος βρίσκεται πλέον στο πεδίο
`user.password`. 

Αν όλα πάνε καλά, εισάγουμε στο αντικείμενο `session` το `id` και το
`username` του χρήστη.

</div>


## Έξοδος

```python
@app.route('/logout')
def logout():
    session.pop('user_id', None)
    session.pop('username', None)
    flash('You were logged out')
    return redirect(url_for('show_entries'))
```

<div class="notes">

Για την έξοδο, αντιστρόφως από την είσοδο, αφαιρούμε από το
αντικείμενο `session` το `id` και το `username` του χρήστη.

</div>


## Βασικό πρότυπο εμφάνισης

* Το βασικό πρότυπο των σελίδων μας θα δίνεται και πάλι από τη σελίδα
  `layout.html` στον κατάλογο `static`.

    ```html
    <!doctype html>
    <title>Flaskr</title>
    <link rel=stylesheet type=text/css
          href="{{ url_for('static', filename='style.css') }}">
    <div class=page>
      <h1>Flaskr</h1>
      <div class=metanav>
      {% if not session.user_id %}
        <a href="{{ url_for('login') }}">log in</a>
      {% else %}
        <a href="{{ url_for('logout') }}">log out</a>
      {% endif %}
      </div>
      {% for message in get_flashed_messages() %}
        <div class=flash>{{ message }}</div>
      {% endfor %}
      {% block body %}{% endblock %}
    </div>
    ```

<div class="notes">

Ο έλεγχος για τον χρήστη πραγματοποιείται πλέον με το πεδίο `user_id`
του αντικειμένου `session`, αντί για το `logged_in` της προηγούμενης
έκδοσης. 

</div>


## Εμφάνιση αναρτήσεων

* Η σελίδα εμφάνισης αναρτήσεων επεκτείνει το βασικό πρότυπο.

* Δημιουργούμε το ακόλουθο αρχείο `show_entries.html` στον κατάλογο
  `static`.


## Κώδικας εμφάνισης

```html
{% extends "layout.html" %}
{% block body %}
  {% if session.user_id %}
    <form action="{{ url_for('add_entry') }}" method=post class=add-entry>
      <dl>
        <dt>Title:
        <dd><input type=text size=30 name=title>
        <dt>Text:
        <dd><textarea name=text rows=5 cols=40></textarea>
        <dd><input type=submit value=Share>
      </dl>
    </form>
  {% endif %}
  <ul class=entries>
  {% for entry in entries %}
    <li><h2>{{ entry.title }}</h2>
      {{ entry.user.username }} {{ entry.datetime }}
      <p>
      {{ entry.text|safe }}
      </p>
  {% else %}
    <li><em>Unbelievable.  No entries here so far</em>
  {% endfor %}
  </ul>
{% endblock %}
```

<div class="notes">

Στη νέα έκδοση εμφανίζουμε, εκτός από τον τίτλο και το περιεχόμενο
κάθε ανάρτησης, το όνομα του χρήστη και τη χρονική στιγμή που έγινε η
ανάρτηση. 

</div>

## Είσοδος

* Η σελίδα εμφάνισης αναρτήσεων επεκτείνει επίσης το βασικό πρότυπο.

* Δημιουργούμε το ακόλουθο αρχείο `login.html` στον κατάλογο
  `static`.

## Κώδικας εισόδου 

```html
{% extends "layout.html" %}
{% block body %}
  <h2>Login</h2>
  {% if error %}
    <p class=error><strong>Error:</strong> {{ error }}
  {% endif %}
  <form action="{{ url_for('login') }}" method=post>
    <dl>
      <dt>Username:
      <dd><input type=text name=username>
      <dt>Password:
      <dd><input type=password name=password>
      <dd><input type=submit value=Login>
    </dl>
  </form>
{% endblock %}
```

<div class="notes">

Αντίστοιχα, η σελίδα `login.html` επεκτείνει τη `layout.html`,
εισάγοντας σε αυτήν τη φόρμα της εισαγωγής του ονόματος και του
κωδικού του χρήστη.

</div>

## Στυλ

* Για να δώσουμε στυλ στην εφαρμογή μας, δημιουργούμε το αρχείο
  `style.css` στον κατάλογο `static`:

    ```css
    body            { font-family: sans-serif; background: #eee; }
    a, h1, h2       { color: #377ba8; }
    h1, h2          { font-family: 'Georgia', serif; margin: 0; }
    h1              { border-bottom: 2px solid #eee; }
    h2              { font-size: 1.2em; }

    .page           { margin: 2em auto; width: 35em; border: 5px solid #ccc;
    padding: 0.8em; background: white; }
    .entries        { list-style: none; margin: 0; padding: 0; }
    .entries li     { margin: 0.8em 1.2em; }
    .entries li h2  { margin-left: -1em; }
    .add-entry      { font-size: 0.9em; border-bottom: 1px solid #ccc; }
    .add-entry dl   { font-weight: bold; }
    .metanav        { text-align: right; font-size: 0.8em; padding: 0.3em;
    margin-bottom: 1em; background: #fafafa; }
    .flash          { background: #cee5F5; padding: 0.5em;
    border: 1px solid #aacbe2; }
    .error          { background: #f0d6d6; padding: 0.5em; }

    ```

## Εκκίνηση

* Τα βήματα για την εκκίνηση της εφαρμογής παραμένουν ως είχαν και
  προηγούμενως. 


# Χειρισμός Κωδικών

## Το πρόβλημα

* Μέχρι τώρα, οι κωδικοί των χρηστών αποθηκεύονται στη βάση έτσι όπως
  ακριβώς δίνονται.

* Αυτό είναι τεράστιο πρόβλημα ασφαλείας.

* Οι κωδικοί πρέπει να αποθηκεύονται με τρόπο τέτοιο ώστε να μην είναι
  δυνατόν να διαπιστωθούν τα περιεχόμενά τους, ακόμα και αν έχουμε
  πλήρη πρόσβαση στη βάση.

* Για το σκοπό αυτό οι κωδικοί κρυπτογραφούνται με κατάλληλη συνάρτητη
  κατακερματισμού. 

## Bcrypt

* Για την κρυπτογράφηση των κωδικών θα χρησιμοποιήσουμε τη συνάρτηση
  [bcrypt](https://en.wikipedia.org/wiki/Bcrypt).

* Η συνάρτηση αυτή παίρνει ως είσοδο μια συμβολοσειρά και μία τυχαία
  σειρά 128 bits που ονομάζεται *αλάτι* (salt).
  
* Με βάση αυτά, παράγει ως έξοδο μια σειρά από 60 bytes από τα οποία δεν
  μπορούμε να βρούμε τι είχε δοθεί ως είσοδος.
  
<div class="notes">

Το αλάτι είναι απαραίτητο για τον εξής λόγο. Κάποιος μπορεί απλώς να
αρχίσει να μαντεύει δημοφιλείς κωδικούς. Αφού όμως στον κωδικό του
χρήστη μπαίνει και το αλάτι, κάποιος θα πρέπει να μαντέψει εκτός από
τον κωδικό και το αλάτι, που είναι πολύ απίθανο.

Όταν θέλουμε να ελέγξουμε έναν κωδικό, τον εισάγουμε στη συνάρτηση
ελέγχου, μαζί με το αλάτι, και περιμένουμε να βρούμε τον
(κρυπτογραφημένο) κωδικό που έχουμε αποθηκεύσει.

</div>


## Εγκατάσταση bcrypt

* Κατ' αρχήν θα πρέπει να εγκαταστήσουμε τη βιβλιοθήκη bcrypt.

    ```bash
    pip install bcrypt
    ```

## Δημιουργία περιβάλλοντος

* Δημιουργείστε την παρακάτω δομή καταλόγων.

    ```
    /flaskr_orm_3
        /static
        /templates
    ```

<div class="notes">

Πρόκειται για τη γνωστή δομή, δεν αλλάζει κάτι εκτός από το όνομα του
βασικού καταλόγου.

</div>

## Φόρμα εγγραφής

* Αυτή τη φορά θα εμπλουτίσουμε την εφαρμογή με τη δυνατότητα online
  εγγραφής χρηστών.

* Η φόρμα εγγραφής θα είναι η ακόλουθη, την οποία θα την αποθηκεύσουμε
  με το όνομα `register.html` στον κατάλογο `templates`.


## Κώδικας φόρμας


```html
{% extends "layout.html" %}
{% block body %}
  <h2>Register</h2>
  {% if error %}
    <p class=error><strong>Error:</strong> {{ error }}
  {% endif %}
  <form action="{{ url_for('register') }}" method=post>
    <dl>
      <dt>Name:
      <dd><input type=text name=name>
      <dt>Surname:
      <dd><input type=text name=surname>
      <dt>email:
      <dd><input type=email name=email>
      <dt>Username:
      <dd><input type=text name=username>
      <dt>Password:
      <dd><input type=password name=password>
      <dt>Repeat Password:
      <dd><input type=password name=repeat-password>
      <dd><input type=submit value=Register>
    </dl>
  </form>
{% endblock %}
```

## Εγγραφή χρηστών

* Ο χειρισμός της εγγραφής θα γίνεται από την ακόλουθη συνάρτηση, η
  οποία θα απαντά σε αιτήσεις στο μονοπάτι `/register`.
  
* Προσέξτε ότι θα πρέπει στο προοίμιο της εφαρμογής να προσθέσετε τη
  γραμμή:
  
    ```python
	from sqlalchemy import exc
	```
  Αυτό θα το χρησιμοποιήσουμε για το χειρισμό λαθών.
  
 
## Κώδικας εγγραφής

```python
@app.route('/register', methods=['GET', 'POST'])
def register():
	error = None
	if request.method == 'POST':
		password = bcrypt.hashpw(request.form['password'].encode('utf8'),
								 bcrypt.gensalt())
		user = User(request.form['username'],
					request.form['name'],
					request.form['surname'],
					request.form['email'],
					password)
		db.session.add(user)
		try:
			db.session.commit()
			session['user_id'] = user.id
			session['username'] = user.username
		except exc.SQLAlchemyError as ex:
			error = 'Error inserting record in the database'
		if not error:
			flash('Registration successful')
			return redirect(url_for('show_entries'))
	return render_template('register.html', error=error)
```

<div class="notes">

Η κρυπτογράφηση του κωδικού του χρήστη γίνεται με την εντολή:

```python
password = bcrypt.hashpw(request.form['password'].encode('utf8'),
                         bcrypt.gensalt())
```

Η πρώτη παράμετρος της συνάρτησης είναι ο κωδικός. Αυτός είναι η
συμβολοσειρά που έχει εισαχθεί από τον χρήστη. Η bcrypt όμως ως είσοδο
παίρνει μια σειρά από bytes, και όχι χαρακτήρες. Συνεπώς μετατρέπουμε
τη συμβολοσειρά σε σειρά από bytes με την κλήση
`request.form['password'].encode('utf8')`. 

Η δεύτερη παράμετρος της συνάρτησης είναι το αλάτι, το οποίο
παράγεται με την κλήση `bcrypt.gensalt()`.

Στη συνέχεια δημιουργούμε το αντικείμενο που θα αντιπροσωπεύει το
χρήστη και το αποθηκεύουμε στη βάση. Υπάρχει πάντα η περίπτωση κάτι
τέτοιο να μη γίνει: για παράδειγμα, κάποιος δίνει `username` το οποίο
ήδη υπάρχει. Αν συμβεί οποιοδήποτε λάθος σχετικό με τη βάση δεδομένων,
θα λάβουμε μία εξαίρεση `SQLAlchemyError`, συνεπώς τη χειριζόμαστε και
εμφανίζουμε κατάλληλο μήνυμα. Μέχρι τώρα αδιαφορούσαμε για τέτοια
προβλήματα, αλλά σιγά-σιγά θα πρέπει να αρχίσουμε να τα
αντιμετωπίζουμε. 

</div>

## Είσοδος

* Η διαδικασία εισόδου του χρήστη θα πρέπει να αλλάξει ελαφρώς,
  προκειμένου να ελέγχεται ο κρυπτογραφημένος κωδικός.

    ```python
    @app.route('/login', methods=['GET', 'POST'])
    def login():
        error = None
        if request.method == 'POST':
            user = User.query.filter_by(username=request.form['username']).first()
            if user is None:
                error = 'Invalid username'
            elif not bcrypt.checkpw(request.form['password'].encode('utf8'),
                                    user.password.encode('utf8')):
                error = 'Invalid password'
            else:
                session['user_id'] = user.id
                session['username'] = user.username
                flash('You have been logged in')
                return redirect(url_for('show_entries'))
        return render_template('login.html', error=error)
    ```

<div class="notes">

Για τον έλεγχο του κωδικού χρησιμοποιούμε την εντολή:

```python
bcrypt.checkpw(request.form['password'].encode('utf8'),
               user.password.encode('utf8'))
```

H συνάρτηση `bcrypt.checkpw` παίρνει ως πρώτη παράμετρο αυτό που
θέλουμε να ελέγξουμε και ως δεύτερη αυτό με το οποίο θα πρέπει να
είναι ίσο. Θέλουμε να ελέγξουμε τον κωδικό που έχει εισάγει ο χρήστης
στη φόρμα, σε σχέση με τον κωδικό που έχει αποθηκευτεί στη βάση.
Επειδή η `bcrypt.checkpw` λειτουργεί με bytes, και όχι με
συμβολοσειρές, θα πρέπει να μετατρέψουμε τις τιμές των παραμέτρων σε
bytes με την `encode('utf8')`.

</div>


## Βασικό πρότυπο εμφάνισης

* Το βασικό πρότυπο εμφάνισης επίσης πρέπει να αλλαχθεί λίγο, ώστε στο
  πάνω μέρος να εμφανίζουμε όταν πρέπει το σύνδεσμο για είσοδο ή για
  εγγραφή.

    ```html
    <!doctype html>
    <title>Flaskr</title>
    <link rel=stylesheet type=text/css
          href="{{ url_for('static', filename='style.css') }}">
    <div class=page>
      <h1>Flaskr</h1>
      <div class=metanav>
      {% if not session.user_id %}
        <a href="{{ url_for('login') }}">log in</a>
        {% if request.path != '/register' %} 
          or
          <a href="{{ url_for('register') }}">register</a>
        {% endif %}
      {% else %}
        <a href="{{ url_for('logout') }}">log out</a>
      {% endif %}
      </div>
      {% for message in get_flashed_messages() %}
        <div class=flash>{{ message }}</div>
      {% endfor %}
      {% block body %}{% endblock %}
    </div>
    ```

<div class="notes">

Συγκεκριμένα, αν ο χρήστης δεν έχει περάσει τη διαδικασία εισόδου και
η σελίδα που θέλει να δει στην εφαρμογή δεν αντιστοιχεί στο μονοπάτι
`/register`, τότε θέλουμε να του παρέχουμε δύο επιλογές, μία για
είσοδο και μία για εγγραφή. Αν όμως αντιστοιχεί στο μονοπάτι
`/register`, τότε θέλουμε να του δώσουμε μόνο τη δυνατότητα εισόδου
(π.χ., γιατί ξαφνικά θυμήθηκε ότι έχει από παλαιότερα κωδικό εισόδου).

</div>

# MySQL

## Γενικά

* Η SQLite είναι μια πολύ βολική λύση, αλλά το πιθανότερο είναι ότι θα
  θέλουμε να χρησιμοποιήσουμε μια άλλη βάση δεδομένων στην παραγωγή.
  
* Για το σκοπό αυτό, θα κάνουμε τις απαραίτητες αλλαγές στην εφαρμογή
  μας ώστε να χρησιμοποιήσει τη βάση δεδομένων MySQL.
  
* Χάρη στο ORM θα διαπιστώσετε ότι οι αλλαγές είναι ελάχιστες.


## Δημιουργία περιβάλλοντος

* Αντιγράψτε τη δομή καταλόγων και τα αρχεία που δημιουργήσατε για το
`flaskr_orm_3` σε έναν κατάλογο `flaskr_orm_4`.

    ```
    /flaskr_orm_4
        /static
        /templates
    ```

* Δεν θα πειράξουμε καθόλου τα υπάρχοντα αρχεία, όπως θα δείτε. Απλώς
  θα προσθέσουμε μόνο ένα.

## Εγκατάσταση MySQL

* Πηγαίνετε στο
  [http://dev.mysql.com/downloads/](http://dev.mysql.com/downloads/)
  και κατεβάστε την τελευταία έκδοση του MySQL Community Server.

* Επίσης, πηγαίνετε στο
  [http://dev.mysql.com/downloads/](http://dev.mysql.com/downloads/)
  και κατεβάστε την τελευταία έκδοση του MySQL Workbench.

## Εγκατάσταση οδηγού

* Για να επικοινωνήσουμε με τη MySQL από προγράμματα Python
  χρειαζόμαστε τον κατάλληλο *οδηγό* (driver).

* Από το
  [Connector/Python](http://dev.mysql.com/downloads/connector/python/)
  επιλέξτε και κατεβάστε την έκδοση "Platform Independent".

* Όταν τον αποσυμπιέσετε, θα πρέπει να ανοίξετε μια γραμμή εντολών και
  να πάτε στον κατάλογο όπου τον αποσυμπιέσατε.
  
* Στον κατάλογο αυτό, θα πρέπει να δώσετε:

    ```bash
	python setup.py install
	```

<div class="notes">

Σε μία επαγγελματική εφαρμογή, θα θέλατε να χρησιμοποιήσετε τον οδηγό
με τις επεκτάσεις της γλώσσας C. Μπορείτε να βρείτε τις σχετικές
οδηγίες [εδώ](https://dev.mysql.com/doc/connector-python/en/connector-python-installation-source.html).

</div>


## Δημιουργία βάσης

* Δημιουργείστε τη βάση `flaskr` εισάγοντας τις ακόλουθες εντολές SQL
  στον MySQL Workbench:

    ```sql
    CREATE DATABASE flaskr CHARACTER SET utf8 COLLATE utf8_general_ci;

    CREATE USER 'flaskr_user'@'localhost' IDENTIFIED BY 'aojkqw3m2t';

    GRANT ALL PRIVILEGES ON flaskr.* TO 'flaskr_user'@'localhost';
    ```

## Ρύθμιση για χρήση MySQL

* Δημιουργείστε στον κατάλογο `flaskr_orm_4` ένα αρχείο `config.py` στο
  οποίο θα εισάγετε την παρακάτω γραμμή:

    ```python
    SQLALCHEMY_DATABASE_URI = 'mysql+mysqlconnector://flaskr_user:aojkqw3m2t@localhost/flaskr'
    ```

* Στη συνέχεια, αν έχετε υπολογιστή MS-Windows, δώστε το παρακάτω στη
  γραμμή εντολών:

    ```
    set FLASKR_SETTINGS=config.py
    ```

* Αν έχετε υπολογιστή Linux ή Mac, δώστε το παρακάτω στη γραμμή
  εντολών:

    ```bash
    export FLASKR_SETTINGS=config.py
    ```

<div class="notes">

Αυτό εξηγεί τη σημασία της γραμμής:

```python
app.config.from_envvar('FLASKR_SETTINGS', silent=True)
```

στο προοίμιο της εφαρμογής μας. Σημαίνει: αν βρεις ένα αρχείο, το
οποίο να δίνεται από τη μεταβλητή περιβάλλοντος `FLASKR_SETTINGS`,
πάρε τα περιεχόμενά του και ενημέρωσε τις ρυθμίσεις της εφαρμογής με
αυτά.

Οπότε αν βρεθεί το αρχείο `config.py`, θα χρησιμοποιηθεί η βάση MySQL.
Αν όχι, θα χρησιμοποιηθεί η SQLite.

</div>

## Δημιουργία πινάκων

* Για να δημιουργήσουμε τους πίνακες της βάσης μας (entries, users)
  μπορούμε πάλι να χρησιμοποιήσουμε τη γραμμή εντολών Python:

    ```python
    >>> from flaskr import db
    >>> db.create_all()
    ```

## Δημιουργία πινάκων

* Εναλλακτικά, μπορούμε να δημιουργήσουμε την αντίστοιχη εντολή,
  προσθέτοντας στο αρχείο `flaskr.py`:

    ```python
    @app.cli.command('initdb')
    def initdb_command():
        """Initializes the database."""
        db.create_all()
        print('Initialized the database.')
    ```

* Οπότε μπορούμε να δώσουμε από τη γραμμή εντολών:

    ```bash
    flask initdb
    ```

* Αυτό θα μπορούσαμε να το είχαμε κάνει προηγουμένως με την SQLite·
  απλώς προτιμούσαμε να χρησιμοποιούμε τη γραμμή εντολών της Python.

## Επόμενα βήματα

* Δεν υπάρχουν!

<div class="notes">

Όπως είδατε, η μόνη αλλαγή που χρειάστηκε ήταν η δημιουργία ενός
αρχείου και μιας μεταβλητής περιβάλλοντος. Δεν άλλαξε τίποτε στην ίδια
την εφαρμογή μας. 

Αν στη συνέχεια θέλουμε να χρησιμοποιήσουμε μια άλλη βάση, όπως SQL
Server, Oracle, PostreSQL, κ.λπ., αρκεί μόνο να αλλάξουμε τα
περιεχόμενα του αρχείου `config.py` και τίποτε άλλο.

</div>
