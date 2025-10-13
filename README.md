# rest_database

const express = require('express');
const sqlite3 = require('sqlite3').verbose();
const app = express();
const PORT = 3000;

app.use(express.json());

const db = new sqlite3.Database('test.db', (err) =>{
    if(err) {
        console.log(err.message);
    }
    else {
        console.log('Az adatbázis kapcsolat létrejött')
    }
});

db.run(`CREATE TABLE IF NOT EXISTS users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    firstName TEXT NOT NULL,
    lastName TEXT NOT NULL,
    City TEXT NOT NULL,
    address TEXT NOT NULL,
    phone TEXT NOT NULL,
    email TEXT NOT NULL,
    gender TEXT NOT NULL 
    )`);

app.post('/api/users', (req, res) => {
    const { firstName, lastName, City, address, phone, email, gender} = req.body

    db.run(`INSERT INTO users (firstName, lastName, City, address, phone, email, gender) VALUES(?,?,?,?,?,?,?)`,
        [firstName, lastName, City, address, phone, email, gender],
        function (err){
            if (err) {
                console.log(err);
                return res.status(500).json({message: 'Hiba történt az adatok rögzítésekor.'})
            }
            else{
                return res.status(201).send({message: 'Az adatok rögzítése sikeres volt.', id: this.lastID, firstName, lastName, City, address, phone, email, gender})
            }
        }
    )
})

app.delete('/api/users/:id', (req, res)=>{
    const {id} = req.params;
    db.run(`DELETE FROM users WHERE id=?`, [id],
        function (err) {
        if (err) {
            return res.status(500).send(err.message);
        }
        res.status(200).json({ message: 'Sikeres adattörlés'});
        }
    )
})


app.put('/api/users/:id', (req, res)=>{
    const {id} = req.params;
    const { firstName, lastName, City, address, phone, email, gender} = req.body;
    const url = 'UPDATE users SET firstName = ?, lastName = ?, City = ?, address = ?, phone = ?, email = ?, gender = ? WHERE id = ?';
    db.run(url, [firstName, lastName, City, address, phone, email, gender, id],
        function (err){
            if (err){
                res.status(500).send(err.message);
            }
            res.status(200).send({message: 'Sikeres adatfrissítés', firstName, lastName, City, address, phone, email, gender});
        } 
    )
})



app.get('/api/users', (req,res) =>{
    db.all('SELECT * FROM users', [], (err, records) =>{
        if (err) {
            console.error(err);
            return res.status(500).json({ message: 'Hiba történt'})
        }
        console.log('Az adatbázisunk sorai:', records);
        return res.status(200).json(records);
    })
    
})



app.listen(PORT, () =>{
    console.log(`A konzol fut a http://localhost:${PORT} webcímen`)
})
