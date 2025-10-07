# Quick Start Guide - OdooPyClient

## Installation

```bash
pip install odoo-pyclient
```

## Basic Usage

### 1. Connect and Authenticate

```python
from odoo_client import OdooPyClient

odoo = OdooPyClient(
    host='http://localhost',
    port=8069,
    database='your_database',
    username='admin',
    password='admin'
)

# Authenticate
result = odoo.authenticate()
print(f"Authenticated! User ID: {result['uid']}")
print(f"Session ID: {odoo.session_id}")  # Store this for later use
```

### 2. Search and Read

```python
# Search for products
products = odoo.search_read(
    model='product.product',
    domain=[['sale_ok', '=', True]],
    fields=['name', 'lst_price'],
    limit=10
)

for product in products:
    print(f"{product['name']}: ${product['lst_price']}")
```

### 3. Create a Record

```python
# Create a partner
partner_id = odoo.create(
    model='res.partner',
    values={
        'name': 'John Doe',
        'email': 'john@example.com',
        'phone': '+1234567890'
    }
)
print(f"Created partner with ID: {partner_id}")
```

### 4. Update a Record

```python
# Update the partner
odoo.update(
    model='res.partner',
    ids=partner_id,
    values={
        'street': '123 Main St',
        'city': 'New York'
    }
)
print("Partner updated!")
```

### 5. Read Records

```python
# Read the partner data
partner_data = odoo.read(
    model='res.partner',
    ids=[partner_id],
    fields=['name', 'email', 'phone', 'street', 'city']
)
print(partner_data)
```

### 6. Delete a Record

```python
# Delete the partner
odoo.delete(model='res.partner', ids=partner_id)
print("Partner deleted!")
```

### 7. Custom RPC Call

```python
# Confirm a sale order
result = odoo.rpc_call(
    endpoint='/web/dataset/call_kw',
    params={
        'model': 'sale.order',
        'method': 'action_confirm',
        'args': [[sale_order_id]],
        'kwargs': {'context': odoo.context}
    }
)
```

## Common Patterns

### Using Session ID for Persistent Connections

```python
# First connection - authenticate and save session
odoo = OdooPyClient(
    host='http://localhost',
    port=8069,
    database='odoo',
    username='admin',
    password='admin'
)
odoo.authenticate()
session_id = odoo.session_id

# Later - reconnect using session_id
odoo = OdooPyClient(
    host='http://localhost',
    port=8069,
    database='odoo',
    session_id=session_id  # No username/password needed
)
```

### Working with Related Records

```python
# Create sale order with order lines
sale_order_id = odoo.create(
    model='sale.order',
    values={
        'partner_id': partner_id,
        'order_line': [
            (0, 0, {  # (0, 0, {...}) means create new record
                'product_id': product_id,
                'product_uom_qty': 2,
                'price_unit': 50.0,
            }),
            (0, 0, {
                'product_id': another_product_id,
                'product_uom_qty': 1,
                'price_unit': 100.0,
            }),
        ],
    }
)
```

### Advanced Search

```python
# Complex domain with AND/OR logic
partners = odoo.search_read(
    model='res.partner',
    domain=[
        '|',  # OR operator
        ['name', 'ilike', 'john'],
        ['email', 'ilike', 'john'],
        ['customer_rank', '>', 0]  # AND with above
    ],
    fields=['name', 'email', 'phone'],
    order='name ASC',
    limit=20
)
```

### Pagination

```python
# Get records in batches
page_size = 100
offset = 0

while True:
    records = odoo.search_read(
        model='product.product',
        domain=[],
        fields=['name'],
        limit=page_size,
        offset=offset
    )
    
    if not records:
        break
    
    # Process records
    for record in records:
        print(record['name'])
    
    offset += page_size
```

### Error Handling

```python
try:
    result = odoo.authenticate()
except Exception as e:
    print(f"Authentication failed: {e}")

try:
    partner_id = odoo.create(
        model='res.partner',
        values={'name': 'Test Partner'}
    )
except Exception as e:
    print(f"Failed to create partner: {e}")
```

## Useful Domain Operators

| Operator | Description | Example |
|----------|-------------|---------|
| `=` | Equals | `['name', '=', 'John']` |
| `!=` | Not equals | `['state', '!=', 'draft']` |
| `>` | Greater than | `['age', '>', 18]` |
| `>=` | Greater or equal | `['price', '>=', 100]` |
| `<` | Less than | `['date', '<', '2024-01-01']` |
| `<=` | Less or equal | `['quantity', '<=', 10]` |
| `like` | Pattern match (case-sensitive) | `['name', 'like', 'John%']` |
| `ilike` | Pattern match (case-insensitive) | `['email', 'ilike', '%@example.com']` |
| `in` | In list | `['id', 'in', [1, 2, 3]]` |
| `not in` | Not in list | `['state', 'not in', ['draft', 'cancel']]` |

## Logical Operators

- `&` - AND (implicit, default)
- `|` - OR
- `!` - NOT

```python
# OR example: name contains 'john' OR email contains 'john'
domain = ['|', ['name', 'ilike', 'john'], ['email', 'ilike', 'john']]

# NOT example: not a customer
domain = ['!', ['customer_rank', '>', 0]]

# Complex: (A OR B) AND C
domain = ['&', '|', ['name', '=', 'A'], ['name', '=', 'B'], ['active', '=', True]]
```

## Tips

1. **Always authenticate** before making any API calls
2. **Store session_id** for persistent connections
3. **Use search_read** instead of search + read for better performance
4. **Batch operations** when working with many records
5. **Handle exceptions** to catch authentication or API errors
6. **Check Odoo logs** if operations fail unexpectedly
7. **Use context** to set language, timezone, or other session parameters

## Resources

- 📚 [Full Documentation](https://github.com/mohamed-helmy/OdooPyClient/blob/main/README.md)
- 🐛 [Report Issues](https://github.com/mohamed-helmy/OdooPyClient/issues)
- 💡 [Examples](https://github.com/mohamed-helmy/OdooPyClient/tree/main/examples)
- 📖 [Odoo API Reference](https://www.odoo.com/documentation/16.0/developer/reference/backend/orm.html)
