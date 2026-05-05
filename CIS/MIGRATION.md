# M12-02 - Terrapay Costs Migration Schema

## Migration entities

Add these entities to the DbContext in `TheraSoft.CISOrchestrator.Core` / EntityFrameworkCore.

```csharp
[Table("PR_TerrapayCosts")]
public class TerrapayCost : FullAuditedEntity<int>
{
    [Required]
    [StringLength(250)]
    public string Description { get; set; }

    [Required]
    [StringLength(20)]
    public string CostType { get; set; }

    [Column(TypeName = "decimal(18,4)")]
    public decimal? FixedFee { get; set; }

    [Column(TypeName = "decimal(18,4)")]
    public decimal? PercentageFee { get; set; }

    public DateTime EffectiveStartDate { get; set; }
    public DateTime? EffectiveEndDate { get; set; }
    public bool IsActive { get; set; } = true;

    [StringLength(500)]
    public string Comments { get; set; }

    // External institution id returned by Terrapay API.
    public long? TerrapayInstId { get; set; }

    // Functional audit fields.
    public DateTime? DeactivatedAt { get; set; }
    public DateTime? ActivatedAt { get; set; }
    public long? DeactivatedByUserId { get; set; }
    public long? ActivatedByUserId { get; set; }

    public int DestinationCountryId { get; set; }
    [ForeignKey("DestinationCountryId")]
    public virtual Country DestinationCountryFk { get; set; }

    public int DestinationCountryCurrencyId { get; set; }
    [ForeignKey("DestinationCountryCurrencyId")]
    public virtual CountryCurrency DestinationCountryCurrencyFk { get; set; }

    public int? TransactionTypeId { get; set; }
    [ForeignKey("TransactionTypeId")]
    public virtual TransactionType TransactionTypeFk { get; set; }

    public int? CisCompanyId { get; set; }
    [ForeignKey("CisCompanyId")]
    public virtual CisCompany CisCompanyFk { get; set; }
}

[Table("HST_TerrapayCosts")]
public class HstTerrapayCost : FullAuditedEntity<long>
{
    [Column(TypeName = "decimal(18,4)")]
    public decimal? FixedFeeInitial { get; set; }

    [Column(TypeName = "decimal(18,4)")]
    public decimal? FixedFeeFinal { get; set; }

    [Column(TypeName = "decimal(18,4)")]
    public decimal? PercentageFeeInitial { get; set; }

    [Column(TypeName = "decimal(18,4)")]
    public decimal? PercentageFeeFinal { get; set; }

    public int TerrapayCostId { get; set; }
    [ForeignKey("TerrapayCostId")]
    public virtual TerrapayCost TerrapayCostFk { get; set; }

    public DateTime? DeactivatedAt { get; set; }
    public DateTime? ActivatedAt { get; set; }
    public long? DeactivatedByUserId { get; set; }
    public long? ActivatedByUserId { get; set; }
}
```

---

## EF Core migration

```csharp
// File: YYYYMMDDHHMMSS_AddTerrapayCosts.cs
public partial class AddTerrapayCosts : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.CreateTable(
            name: "PR_TerrapayCosts",
            columns: table => new
            {
                Id = table.Column<int>(nullable: false)
                    .Annotation("SqlServer:Identity", "1, 1"),
                Description = table.Column<string>(maxLength: 250, nullable: false),
                CostType = table.Column<string>(maxLength: 20, nullable: false),
                FixedFee = table.Column<decimal>(type: "decimal(18,4)", nullable: true),
                PercentageFee = table.Column<decimal>(type: "decimal(18,4)", nullable: true),
                EffectiveStartDate = table.Column<DateTime>(nullable: false),
                EffectiveEndDate = table.Column<DateTime>(nullable: true),
                IsActive = table.Column<bool>(nullable: false, defaultValue: true),
                Comments = table.Column<string>(maxLength: 500, nullable: true),
                TerrapayInstId = table.Column<long>(nullable: true),
                DestinationCountryId = table.Column<int>(nullable: false),
                DestinationCountryCurrencyId = table.Column<int>(nullable: false),
                TransactionTypeId = table.Column<int>(nullable: true),
                CisCompanyId = table.Column<int>(nullable: true),
                DeactivatedAt = table.Column<DateTime>(nullable: true),
                ActivatedAt = table.Column<DateTime>(nullable: true),
                DeactivatedByUserId = table.Column<long>(nullable: true),
                ActivatedByUserId = table.Column<long>(nullable: true),
                CreationTime = table.Column<DateTime>(nullable: false),
                CreatorUserId = table.Column<long>(nullable: true),
                LastModificationTime = table.Column<DateTime>(nullable: true),
                LastModifierUserId = table.Column<long>(nullable: true),
                IsDeleted = table.Column<bool>(nullable: false, defaultValue: false),
                DeleterUserId = table.Column<long>(nullable: true),
                DeletionTime = table.Column<DateTime>(nullable: true),
            },
            constraints: table =>
            {
                table.PrimaryKey("PK_PR_TerrapayCosts", x => x.Id);
                table.ForeignKey(
                    name: "FK_PR_TerrapayCosts_CAT_Countries_DestinationCountryId",
                    column: x => x.DestinationCountryId,
                    principalTable: "CAT_Countries",
                    principalColumn: "Id",
                    onDelete: ReferentialAction.Restrict);
                table.ForeignKey(
                    name: "FK_PR_TerrapayCosts_CAT_CountryCurrencies_DestinationCountryCurrencyId",
                    column: x => x.DestinationCountryCurrencyId,
                    principalTable: "CAT_CountryCurrencies",
                    principalColumn: "Id",
                    onDelete: ReferentialAction.Restrict);
                table.ForeignKey(
                    name: "FK_PR_TerrapayCosts_CAT_TransactionTypes_TransactionTypeId",
                    column: x => x.TransactionTypeId,
                    principalTable: "CAT_TransactionTypes",
                    principalColumn: "Id",
                    onDelete: ReferentialAction.Restrict);
                table.ForeignKey(
                    name: "FK_PR_TerrapayCosts_CisCompanies_CisCompanyId",
                    column: x => x.CisCompanyId,
                    principalTable: "CisCompanies",
                    principalColumn: "Id",
                    onDelete: ReferentialAction.Restrict);
            });

        migrationBuilder.CreateIndex(
            name: "IX_PR_TerrapayCosts_DestinationCountryId",
            table: "PR_TerrapayCosts",
            column: "DestinationCountryId");

        migrationBuilder.CreateIndex(
            name: "IX_PR_TerrapayCosts_DestinationCountryCurrencyId",
            table: "PR_TerrapayCosts",
            column: "DestinationCountryCurrencyId");

        migrationBuilder.CreateIndex(
            name: "IX_PR_TerrapayCosts_TransactionTypeId",
            table: "PR_TerrapayCosts",
            column: "TransactionTypeId");

        migrationBuilder.CreateIndex(
            name: "IX_PR_TerrapayCosts_CisCompanyId",
            table: "PR_TerrapayCosts",
            column: "CisCompanyId");

        migrationBuilder.CreateIndex(
            name: "IX_PR_TerrapayCosts_IsActive_EffectiveStartDate",
            table: "PR_TerrapayCosts",
            columns: new[] { "IsActive", "EffectiveStartDate" });

        migrationBuilder.CreateIndex(
            name: "IX_PR_TerrapayCosts_UniqueBusinessRule",
            table: "PR_TerrapayCosts",
            columns: new[]
            {
                "DestinationCountryId",
                "DestinationCountryCurrencyId",
                "TransactionTypeId",
                "TerrapayInstId",
                "CostType"
            },
            unique: true,
            filter: "[IsDeleted] = 0");

        migrationBuilder.CreateTable(
            name: "HST_TerrapayCosts",
            columns: table => new
            {
                Id = table.Column<long>(nullable: false)
                    .Annotation("SqlServer:Identity", "1, 1"),
                FixedFeeInitial = table.Column<decimal>(type: "decimal(18,4)", nullable: true),
                FixedFeeFinal = table.Column<decimal>(type: "decimal(18,4)", nullable: true),
                PercentageFeeInitial = table.Column<decimal>(type: "decimal(18,4)", nullable: true),
                PercentageFeeFinal = table.Column<decimal>(type: "decimal(18,4)", nullable: true),
                TerrapayCostId = table.Column<int>(nullable: false),
                DeactivatedAt = table.Column<DateTime>(nullable: true),
                ActivatedAt = table.Column<DateTime>(nullable: true),
                DeactivatedByUserId = table.Column<long>(nullable: true),
                ActivatedByUserId = table.Column<long>(nullable: true),
                CreationTime = table.Column<DateTime>(nullable: false),
                CreatorUserId = table.Column<long>(nullable: true),
                LastModificationTime = table.Column<DateTime>(nullable: true),
                LastModifierUserId = table.Column<long>(nullable: true),
                IsDeleted = table.Column<bool>(nullable: false, defaultValue: false),
                DeleterUserId = table.Column<long>(nullable: true),
                DeletionTime = table.Column<DateTime>(nullable: true),
            },
            constraints: table =>
            {
                table.PrimaryKey("PK_HST_TerrapayCosts", x => x.Id);
                table.ForeignKey(
                    name: "FK_HST_TerrapayCosts_PR_TerrapayCosts_TerrapayCostId",
                    column: x => x.TerrapayCostId,
                    principalTable: "PR_TerrapayCosts",
                    principalColumn: "Id",
                    onDelete: ReferentialAction.Cascade);
            });

        migrationBuilder.CreateIndex(
            name: "IX_HST_TerrapayCosts_TerrapayCostId",
            table: "HST_TerrapayCosts",
            column: "TerrapayCostId");
    }

    protected override void Down(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.DropTable(name: "HST_TerrapayCosts");
        migrationBuilder.DropTable(name: "PR_TerrapayCosts");
    }
}
```

---

## Verified foreign keys against existing backend tables

Verified in the current backend model:

- `Country` -> `CAT_Countries`
- `CountryCurrency` -> `CAT_CountryCurrencies`
- `CisCompany` -> `CisCompanies`
- `TransactionType` catalog available in current migrations as `CAT_TransactionTypes`

`TerrapayInstId` is intentionally **not** defined as a local FK because it must store the external institution identifier returned by Terrapay API (banks / wallets) and later be matched against that external endpoint response.

---

## Application layer DTOs

```csharp
public class GetTerrapayCostsInput : PagedAndSortedResultRequestDto
{
    public string Filter { get; set; }
    public string CostType { get; set; }
    public int? DestinationCountryId { get; set; }
    public int? DestinationCountryCurrencyId { get; set; }
    public int? TransactionTypeId { get; set; }
    public long? TerrapayInstId { get; set; }
    public int? ActiveFilter { get; set; }
}

public class TerrapayCostDto : EntityDto<int>
{
    public string Description { get; set; }
    public string CostType { get; set; }
    public decimal? FixedFee { get; set; }
    public decimal? PercentageFee { get; set; }
    public DateTime EffectiveStartDate { get; set; }
    public DateTime? EffectiveEndDate { get; set; }
    public bool IsActive { get; set; }
    public string Comments { get; set; }
    public int DestinationCountryId { get; set; }
    public string DestinationCountryName { get; set; }
    public int DestinationCountryCurrencyId { get; set; }
    public string DestinationCurrencyCode { get; set; }
    public int? TransactionTypeId { get; set; }
    public string TransactionTypeName { get; set; }
    public long? TerrapayInstId { get; set; }
}

public class CreateOrEditTerrapayCostDto : EntityDto<int?>
{
    [Required]
    [StringLength(250)]
    public string Description { get; set; }

    [Required]
    [StringLength(20)]
    public string CostType { get; set; }

    public decimal? FixedFee { get; set; }
    public decimal? PercentageFee { get; set; }

    [Required]
    public DateTime EffectiveStartDate { get; set; }
    public DateTime? EffectiveEndDate { get; set; }
    public bool IsActive { get; set; }
    public string Comments { get; set; }

    [Required]
    public int DestinationCountryId { get; set; }

    [Required]
    public int DestinationCountryCurrencyId { get; set; }

    public int? TransactionTypeId { get; set; }
    public int? CisCompanyId { get; set; }
    public long? TerrapayInstId { get; set; }
}
```

---

## Functional notes

### Activate / deactivate

- Deactivate: set `IsDeleted = 1`, `IsActive = 0`, `DeactivatedByUserId`, `DeactivatedAt`.
- Activate: rollback logical delete with `IsDeleted = 0`, `IsActive = 1`, `ActivatedByUserId`, `ActivatedAt`.

### Import

- Validate file format and required columns.
- Validate duplicate business rule by functional key, independently of fee values.
- Insert one history row for every inserted / updated record.

### Export

- Export visible rows.
- Include report header with active filters.

### Template download

- Include required columns in English/internal structure.
- Include one sample row.

---

## NSwag

Once backend is implemented:

```bash
cd nswag && refresh.bat
```

Then replace the frontend mock service with generated proxies.
