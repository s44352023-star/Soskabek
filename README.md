💻 Kod (HTML + CSS)
HTML
<!DOCTYPE html>
<html lang="uz">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Chiroyli Narxlar Jadvali</title>
    <style>
        /* Umumiy sahifa dizayni */
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #f4f7f6;
            color: #333;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            margin: 0;
        }

        /* Jadval konteyneri */
        .table-container {
            width: 90%;
            max-width: 900px;
            background: #ffffff;
            padding: 20px;
            border-radius: 12px;
            box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
            overflow-x: auto;
        }

        /* Jadval asosiy qismi */
        .pricing-table {
            width: 100%;
            border-collapse: collapse;
            text-align: left;
            font-size: 15px;
        }

        /* Sarlavha va Pastki qism umumiy bezagi */
        .pricing-table th, 
        .pricing-table td {
            padding: 14px 18px;
            border-bottom: 1px solid #e0e0e0;
        }

        /* thead dizayni */
        .pricing-table thead th {
            background-color: #2c3e50;
            color: #ffffff;
            font-weight: 600;
            text-transform: uppercase;
            font-size: 13px;
            letter-spacing: 0.5px;
        }

        .pricing-table thead th:first-child {
            border-top-left-radius: 8px;
        }

        .pricing-table thead th:last-child {
            border-top-right-radius: 8px;
        }

        /* Zebra striping (juft qatorlarni bo'yash) */
        .pricing-table tbody tr:nth-child(even) {
            background-color: #f9fbfb;
        }

        /* Hover effect (sichqoncha borgandagi holat) */
        .pricing-table tbody tr:hover {
            background-color: #f1f8ff;
            transition: background-color 0.2s ease-in-out;
        }

        /* Matn tekislash va shriftlar */
        .pricing-table td.price,
        .pricing-table td.qty,
        .pricing-table td.total {
            text-align: right;
        }

        .pricing-table th.price,
        .pricing-table th.qty,
        .pricing-table th.total {
            text-align: right;
        }

        /* tfoot dizayni */
        .pricing-table tfoot td {
            font-weight: bold;
            background-color: #ecf0f1;
            color: #2c3e50;
        }

        .pricing-table tfoot tr:last-child td:first-child {
            border-bottom-left-radius: 8px;
        }

        .pricing-table tfoot tr:last-child td:last-child {
            border-bottom-right-radius: 8px;
        }

        /* Maxsus urg'u uchun */
        .category-row {
            background-color: #e8f4fd !important;
            font-weight: bold;
            color: #1d6fa5;
        }
    </style>
</head>
<body>

<div class="table-container">
    <table class="pricing-table">
        <thead>
            <tr>
                <th>Nom</th>
                <th>Tavsif</th>
                <th class="price">Narx</th>
                <th class="qty">Soni</th>
                <th class="total">Jami</th>
            </tr>
        </thead>
        <tbody>
            <!-- Rowspan misoli uchun kategoriya -->
            <tr class="category-row">
                <td colspan="5">Kategoriya: Veb Dasturlash xizmatlari</td>
            </tr>
            <tr>
                <td>Landing Page</td>
                <td>Bir sahifali zamonaviy veb-sayt dizayni va dasturlashi</td>
                <td class="price">$300</td>
                <td class="qty">1</td>
                <td class="total">$300</td>
            </tr>
            <tr>
                <td>E-commerce</td>
                <td>To'liq internet do'kon, to'lov tizimlari bilan</td>
                <td class="price">$1,200</td>
                <td class="qty">1</td>
                <td class="total">$1,200</td>
            </tr>
            <tr>
                <td>Texnik yordam</td>
                <td>Har oylik sayt xavfsizligi va yangilanishi</td>
                <td class="price">$50</td>
                <td class="qty">3</td>
                <td class="total">$150</td>
            </tr>
            <tr>
                <td>SEO Optimization</td>
                <td>Qidiruv tizimlarida birinchi o'rinlarga chiqarish</td>
                <td class="price">$200</td>
                <td class="qty">2</td>
                <td class="total">$400</td>
            </tr>
        </tbody>
        <tfoot>
            <tr>
                <td colspan="4" style="text-align: right;">Umumiy Hisob:</td>
                <td class="total">$2,050</td>
            </tr>
        </tfoot>
    </table>
</div>

</body>
</html>
