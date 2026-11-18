
from fastapi.testclient import TestClient
from main import app

client = TestClient(app)


def test_root():
    response = client.get("/")
    assert response.status_code == 200
    assert response.json() == {
        "message": "Welcome to the FastAPI application!"
    }


def test_read_item():
    response = client.get("/items/10?q=hello")
    assert response.status_code == 200
    assert response.json() == {
        "item_id": 10,
        "query_param": "hello"
    }


def test_create_item():
    response = client.post(
        "/items/",
        json={
            "name": "Laptop",
            "description": "Test laptop",
            "price": 1000,
            "tax": 100
        }
    )

    assert response.status_code == 200
    assert response.json()["total_price"] == 1100


def test_invalid_item():
    response = client.post(
        "/items/",
        json={"name": "Laptop"}
    )

    assert response.status_code == 422
