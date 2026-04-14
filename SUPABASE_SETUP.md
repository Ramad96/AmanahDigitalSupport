# Supabase — AmanahDigital Support

Database backend for `support.amanahdigital.co.uk`. Submissions are stored in the `support` schema.

---

## Viewing submissions

Open the Supabase dashboard, go to **Table Editor**, select the **support** schema from the schema dropdown, then open **support_feedback**.

### Useful queries

Run these in the **SQL Editor**:

```sql
-- Most recent 50 submissions
select created_at, type, product, email, subject, message
from support.support_feedback
order by created_at desc
limit 50;

-- Filter by product
select * from support.support_feedback
where product = 'Noor: A Better Ramadan';

-- Count by submission type
select type, count(*)
from support.support_feedback
group by type;

-- Count by product
select product, count(*)
from support.support_feedback
group by product
order by count desc;
```

---

## Table structure

| Column       | Type        | Notes                                              |
|--------------|-------------|----------------------------------------------------|
| `id`         | uuid        | Auto-generated primary key                         |
| `created_at` | timestamptz | Set automatically on insert                        |
| `type`       | text        | `feedback`, `feature_request`, or `bug_report`     |
| `product`    | text        | Product name or `General`                          |
| `email`      | text        | Submitter's email address                          |
| `subject`    | text        | Brief summary from the form                        |
| `message`    | text        | Full details from the form                         |

---

## Email notifications (optional)

To receive an email on every new submission, set up a Database Webhook:

1. Go to **Project Settings → Database → Webhooks**.
2. Create a new webhook on the `support.support_feedback` table, trigger: **INSERT**.
3. Point it at a service such as [Resend](https://resend.com), [Postmark](https://postmarkapp.com), or [Zapier](https://zapier.com).
