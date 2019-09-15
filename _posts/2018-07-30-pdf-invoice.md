---
title: Automating Invoice Generation - Python
layout: post
icon : fa-code 
---

Every process can benefit from automation. For large industries to small business owners even for personal workflow automation plays an important part. In this post we look at generating invoices that can be applied to any form to trade. We will generate the invoice of the items and their payable dues. The invoice will be generated from a HTML template to PDF form because PDF is a widely used format that can be printed or transferred with ease.

## Creating an HTML template  

As this is not a static HTML file and requires to be created for each transaction accordingly, we use [Jinja](http://jinja.pocoo.org/). Jinja is a templating engine for python which allows for dynamic creation of HTML documents.  
First we need to populate a table with a list of all the items and calculate their costs accordingly. We use data stored in a CSV file and read it using [Pandas](https://pandas.pydata.org/).

```python
items = pandas.read_csv("items.csv")
```

Here the `items` is a `DataFrame` object which stores the file's data in a tabular format.  
Now we need that data to populate the HTML table. We do that by adding a Jinja executable block to the HTML document.  
__NOTE__ Ignore the `\` characters, they are to render markdown( for me ;) )

```html
\{\% for i in range(items.shape[0]) %}
<tr>
    <td>
        \{\{ items["name"][i] }}
    </td>
    <td>
        \{\{ items["id"][i] }}
    </td>
    <td>
        \{\{ items["quantity"][i] }}
    </td>
    <td>
        \{\{ items["price"][i] }}
    </td>
    <td>
        \{\{ items["price"][i] * items["quantity"][i] }}
    </td>
</tr>
\{\% endfor %}
```

Here the code in `\{\% %}` is Jinja executable block i.e. the blocks will execute. Here they execute the for loop and use the `\{\{ }}` block to fill in with variables.  
Finally the template is executed with Jinja and all the required variables are passed to it.

```python
with open("invoice.html") as html_template:
    html = html_template.read()
    temp = Template(html)
# Render Jinja blocks
out = temp.render(items=items)
```

This reads the HTML file template with the jinja blocks. Then the template is loaded into Jinja and rendered by passing all the required variables to it. Here the `out` is a HTML document.

## Rendering HTML to PDF  

Now that we have our final HTML document, we need to convert it to PDF. For that we use [WeasyPrint](https://weasyprint.org/), it provides a simple interface with a high degree of control over the output.

```python
from weasyprint import HTML, CSS

HTML(string=out).write_pdf(
    "invoice.pdf", stylesheets=["style.css",]
)
```

We can also attach a CSS file to create our desired styles. Also in the CSS file we can use the `@page` selector to select the properties of the page in PDF.

```css
@page{
    size: A4;
    margin: 0.5cm;
}
```

Now this PDF file is ready for print or transmission or whatever purpose we see require.
