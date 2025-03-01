import sqlite3
import tkinter as tk
from tkinter import messagebox

# Création de la base de données SQLite
conn = sqlite3.connect("billetterie.db")
cursor = conn.cursor()

# Création de la table des billets
cursor.execute("""
CREATE TABLE IF NOT EXISTS billets (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    evenement TEXT NOT NULL,
    prix REAL NOT NULL,
    stock INTEGER NOT NULL
)
""")
conn.commit()

# Insertion des billets de test si la table est vide
cursor.execute("SELECT COUNT(*) FROM billets")
if cursor.fetchone()[0] == 0:
    cursor.executemany("INSERT INTO billets (evenement, prix, stock) VALUES (?, ?, ?)",
                       [("Concert Pop", 50.0, 100), ("Match de Foot", 30.0, 200), ("Théâtre", 20.0, 50)])
    conn.commit()

# Fonction pour afficher les billets disponibles
def afficher_billets():
    cursor.execute("SELECT * FROM billets")
    billets = cursor.fetchall()
    text_output.delete("1.0", tk.END)
    for billet in billets:
        text_output.insert(tk.END, f"ID: {billet[0]} | Événement: {billet[1]} | Prix: {billet[2]}$ | Stock: {billet[3]}\n")

# Fonction pour acheter un billet
def acheter_billet():
    try:
        billet_id = int(entry_id.get())
        quantite = int(entry_quantite.get())

        cursor.execute("SELECT stock FROM billets WHERE id=?", (billet_id,))
        resultat = cursor.fetchone()

        if resultat and resultat[0] >= quantite:
            nouveau_stock = resultat[0] - quantite
            cursor.execute("UPDATE billets SET stock=? WHERE id=?", (nouveau_stock, billet_id))
            conn.commit()
            messagebox.showinfo("Achat réussi", f"Vous avez acheté {quantite} billet(s) pour l'événement {billet_id}.")
            afficher_billets()
        else:
            messagebox.showwarning("Stock insuffisant", "Il n'y a pas assez de billets disponibles.")

    except ValueError:
        messagebox.showerror("Erreur", "Veuillez entrer un ID et une quantité valides.")

# Interface utilisateur avec Tkinter
root = tk.Tk()
root.title("Billetterie")

frame = tk.Frame(root)
frame.pack(pady=10)

tk.Label(frame, text="ID du billet:").grid(row=0, column=0)
entry_id = tk.Entry(frame)
entry_id.grid(row=0, column=1)

tk.Label(frame, text="Quantité:").grid(row=1, column=0)
entry_quantite = tk.Entry(frame)
entry_quantite.grid(row=1, column=1)

btn_acheter = tk.Button(frame, text="Acheter", command=acheter_billet)
btn_acheter.grid(row=2, columnspan=2, pady=10)

text_output = tk.Text(root, height=10, width=50)
text_output.pack(pady=10)

btn_afficher = tk.Button(root, text="Afficher les billets", command=afficher_billets)
btn_afficher.pack()

# Chargement initial des billets
afficher_billets()

root.mainloop()

# Fermeture de la connexion
conn.close()
