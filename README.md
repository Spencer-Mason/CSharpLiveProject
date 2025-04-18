# CSharpLiveProject
Code Summary of The Tech Academy's C# &amp; .NET Live Project

# Theatre CMS – Rental Histories

An ASP .NET MVC & Entity Framework “code‑first” CMS module for a theatre/acting company website.  
Specifically scaffolds and styles the **Rental Histories** section so non‑technical users can effortlessly **create**, **view**, **edit**, **delete**, and **sort** rental records.

---

## 📋 Table of Contents

1. [Project Overview](#project-overview)  
2. [Tech Stack & Workflow](#tech-stack--workflow)  
3. [Key Responsibilities](#key-responsibilities)  
4. [Code Highlights](#code-highlights)  
   - Model Definition  
   - Scaffolding & Migrations  
   - Create/Edit Pages + Label Toggle (▶️ Animated demo below)  
   - Index Page Enhancements (Icons, Ellipses Menu)  
   - Sorting Feature (DateCreated, Damaged, A–Z)  
   - Controller Redirect Bugfix  
5. [Project Workflow](#project-workflow)  

---

## 🧾 Project Overview

This module powers the **Rental Histories** area of a theatre company’s CMS. Users can:

- **Log new rentals** with details (`Rental`, `Damaged?`, `Notes`), time‑stamped via a **DateCreated** property.  
- **Browse** all rentals in a responsive table, with visual cues (green check or red 'X').  
- **Inline edit** label text (“Notes” ↔ “Damages Incurred”) based on the “Damaged?” checkbox.  
  ![Label Toggle Animation](RentalHistoryTextChange.gif)  
- **Hover** to reveal an ellipses icon for each row, opening a mini‑menu: Edit, Details, Delete.  
- **Sort** by creation date (newest), damaged status, and rental name (A–Z, Z–A).

---

## 🛠 Tech Stack & Workflow

- **Framework:** ASP .NET MVC (Razor views)  
- **ORM:** Entity Framework (Code‑First)  
- **Front‑End:** HTML5, CSS3, Bootstrap, Font Awesome, vanilla JavaScript / jQuery  
- **Dev Tools:** Visual Studio, Package Manager Console, Git/Azure DevOps  

---

## 🚀 Key Responsibilities

- **Model & Migration**  
  - Created `RentalHistory` model with auto‑timestamping (`DateCreated`).  
  - Generated and applied EF migration via PMC.

- **Scaffolding CRUD**  
  - Used built‑in MVC scaffolding to produce **Create**, **Edit**, **Details**, **Delete**, **Index** views and controller.

- **Custom Styling & Scripts**  
  - Applied customer color scheme via `Rent.css`.  
  - Added JS to toggle label text on Create/Edit pages.  
  - Enhanced Index view with Font Awesome icons, hover ellipses, contextual menus, and dynamic row styling.

- **Sorting Logic**  
  - Extended Index page to sort by `DateCreated` (default), damaged status, and rental name.

- **Bug Fix**  
  - Added a dummy `RentalsController` to redirect legacy home‑page link to `RentalItemsController`.

---

## 🔍 Code Highlights

### 1. Model Definition

```csharp
// Areas/Rent/Models/RentalHistory.cs
public class RentalHistory
{
    public int RentalHistoryID { get; set; }

    [Display(Name = "Damaged?")]
    public bool RentalDamaged { get; set; }

    [Display(Name = "Notes")]
    public string DamagesIncurred { get; set; }

    public string Rental { get; set; }
    public DateTime DateCreated { get; set; }

    public RentalHistory() => DateCreated = DateTime.Now;
}
```

---

### 2. Scaffolding & Migrations

```powershell
Add-Migration AddRentalHistory
Update-Database
# Scaffold via “Add > New Scaffolded Item > MVC 5 Controller with views…”
```

---

### 3. Create/Edit Pages & Label Toggle

```html
@model TheatreCMS3.Areas.Rent.Models.RentalHistory

<h2>Create Rental History</h2>
<link href="~/Content/Areas/Rent.css" rel="stylesheet" />

@using (Html.BeginForm()) {
  <div class="form-horizontal rental-historiesCreate--form">
    @Html.LabelFor(m => m.Rental)
    @Html.EditorFor(m => m.Rental)

    @Html.LabelFor(m => m.RentalDamaged)
    @Html.EditorFor(m => m.RentalDamaged, new { id = "RentalDamaged" })

    @Html.LabelFor(m => m.DamagesIncurred, htmlAttributes: new { id = "DamagesIncurredLabel" })
    @Html.EditorFor(m => m.DamagesIncurred, new { htmlAttributes = new { id = "DamagesIncurred" } })

    <input type="submit" value="Create" class="btn btn-default" />
  </div>
}

@section Scripts {
  @Scripts.Render("~/bundles/jqueryval")
  <script src="~/Areas/Rent/Scripts/Rent.js"></script>
}
```

```javascript
// Areas/Rent/Scripts/Rent.js
document.addEventListener('DOMContentLoaded', () => {
  const chk = document.getElementById('RentalDamaged'),
        lbl = document.getElementById('DamagesIncurredLabel');

  function updateLabel() {
    lbl.textContent = chk.checked ? 'Damages Incurred' : 'Notes';
  }

  chk.addEventListener('change', updateLabel);
  updateLabel();
});
```

---

### 4. Index Page Enhancements

```html
@* Areas/Rent/Views/RentalItems/Index.cshtml *@
<table class="rental-historiesIndex--historiesTbl">
  <thead>
    <tr>
      <th>Rental</th>
      <th>Damaged?</th>
      <th>Notes</th>
      <th>Actions</th>
    </tr>
  </thead>
  <tbody>
    @foreach(var item in Model)
    {
      <tr class="rental-historiesIndex--row">
        <td><span class="badge">@item.Rental</span></td>
        <td>
          @if(item.RentalDamaged)
            <i class="fa fa-times-circle"></i>
          else
            <i class="fa fa-check-circle"></i>
        </td>
        <td class="@(item.RentalDamaged ? "damaged" : "notDamaged")">
          @item.DamagesIncurred
        </td>
        <td class="actions">
          <span class="rental-historiesIndex--ellipsis">…</span>
          <div class="rental-historiesIndex--menu">
            @Html.ActionLink("Edit", "Edit", new { id = item.RentalHistoryID })
            @Html.ActionLink("Details", "Details", new { id = item.RentalHistoryID })
            @Html.ActionLink("Delete", "Delete", new { id = item.RentalHistoryID })
          </div>
        </td>
      </tr>
    }
  </tbody>
</table>
<script src="~/Areas/Rent/Scripts/Rent.js"></script>
```

---

### 5. Sorting Feature

```javascript
// In Areas/Rent/Scripts/Rent.js
document.getElementById('sortSelector').addEventListener('change', function() {
  const container = document.querySelector('.rental-historiesIndex--historiesTbl tbody');
  const rows = Array.from(container.children);
  // sort logic: by DateCreated, RentalDamaged, or text
  // re-append sorted rows back to the tbody
});
```

---

### 6. Controller Redirect Bugfix

```csharp
// Areas/Rent/Controllers/RentalsController.cs
public class RentalsController : Controller
{
    public ActionResult Index()
        => RedirectToAction("Index", "RentalItems", new { area = "Rent" });
}
```

---

## 🧩 Project Workflow

1. **Model & Migration** — Defined model, ran EF migration.  
2. **Scaffolded CRUD** — Generated Razor views & controller.  
3. **Styling & Icons** — Applied CSS, Font Awesome.  
4. **JavaScript** — Label toggle, ellipses menu, sorting.  
5. **Bug Fixes** — Redirect controller, routing corrections.  

