from flask import Flask, request, jsonify

app = Flask(__name__)

users = []           
current_id = 1           


def find_user(user_id: int):
    """Retorna o dicionário do usuário ou None."""
    return next((u for u in users if u["id"] == user_id), None)


@app.post("/users")
def create_user():
    
    data = request.get_json(silent=True)

    if not data:
        return jsonify({"error": "JSON inválido ou ausente"}), 400

    nome = data.get("nome")
    email = data.get("email")

    
    if not isinstance(nome, str) or not isinstance(email, str):
        return jsonify({"error": "Campos 'nome' e 'email' são obrigatórios (string)."}), 400

    global current_id
    user = {"id": current_id, "nome": nome.strip(), "email": email.strip()}
    users.append(user)
    current_id += 1

    return jsonify(user), 201 


@app.get("/users")
def read_users():
    
    return jsonify(users), 200


@app.get("/users/<int:user_id>")
def read_single_user(user_id: int):
    user = find_user(user_id)
    if user is None:
        return jsonify({"error": "Usuário não encontrado"}), 404
    return jsonify(user), 200


@app.put("/users/<int:user_id>")
def update_user(user_id: int):
    user = find_user(user_id)
    if user is None:
        return jsonify({"error": "Usuário não encontrado"}), 404

    data = request.get_json(silent=True)
    if not data:
        return jsonify({"error": "JSON inválido ou ausente"}), 400

    
    if "nome" in data:
        if not isinstance(data["nome"], str):
            return jsonify({"error": "Campo 'nome' deve ser string."}), 400
        user["nome"] = data["nome"].strip()

    if "email" in data:
        if not isinstance(data["email"], str):
            return jsonify({"error": "Campo 'email' deve ser string."}), 400
        user["email"] = data["email"].strip()

    return jsonify(user), 200


@app.delete("/users/<int:user_id>")
def delete_user(user_id: int):
    user = find_user(user_id)
    if user is None:
        return jsonify({"error": "Usuário não encontrado"}), 404

    users.remove(user)
    return jsonify({"message": "Usuário excluído com sucesso"}), 200


if __name__ == "__main__":
    
    app.run(debug=True)
