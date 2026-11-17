# DAX ölçüləri

Bütün ölçülər Power BI modelində qovluqlara (Display folder) bölünüb. Ölçü adları Azərbaycan dilindədir.

## Ümumi DAX

```dax
Satış cəmi = SUM ( 'Append'[Satış] )
```

```dax
Ümumi müştəri sayı = DISTINCTCOUNT ( 'Append'[Şəhər_İD] )
```

```dax
Maşın sayı (növ) = DISTINCTCOUNT ( 'Append'[Maşın kodu] )
```

## Okeaniya DAX (stok)

Stok gündəlik qalıqdır (snapshot), ona görə tarixlər üzrə toplanmır.

```dax
Total Stok sayı = SUM ( 'Append (2)'[Stock] )
```

**Son stok sayı** hər maşının son qeyd olunan stokunu götürüb cəmləyir:

```dax
son stok sayı =
SUMX (
    VALUES ( 'Toyota_sku'[Maşın kodu] ),
    CALCULATE (
        SUM ( 'Append (2)'[Stock] ),
        LASTNONBLANK ( 'Calendar'[Date], CALCULATE ( SUM ( 'Append (2)'[Stock] ) ) )
    )
)
```

## Avropa DAX (müqayisə ölçüləri)

Bu ölçülər **seçimdəki son tarixi** əsas götürür. Beləliklə, il və ya ay seçilməyəndə də məntiqli nəticə verir (son il ilə əvvəlki il, son ay ilə əvvəlki ay müqayisə olunur).

```dax
Cari il satışı =
VAR y = YEAR ( CALCULATE ( MAX ( 'Append'[Tarix] ) ) )
RETURN
    CALCULATE (
        [Satış cəmi],
        FILTER ( VALUES ( 'Calendar'[Date] ), YEAR ( 'Calendar'[Date] ) = y )
    )
```

```dax
Ötən ilin eyni dövrü satışı =
VAR y = YEAR ( CALCULATE ( MAX ( 'Append'[Tarix] ) ) )
RETURN
    CALCULATE (
        [Satış cəmi],
        SAMEPERIODLASTYEAR (
            FILTER ( VALUES ( 'Calendar'[Date] ), YEAR ( 'Calendar'[Date] ) = y )
        ),
        REMOVEFILTERS ( 'Calendar' )
    )
```

```dax
İldən ilə böyümə % =
DIVIDE ( [Cari il satışı] - [Ötən ilin eyni dövrü satışı], [Ötən ilin eyni dövrü satışı] )
```

```dax
Cari ay satışı =
VAR d = CALCULATE ( MAX ( 'Append'[Tarix] ) )
RETURN
    CALCULATE (
        [Satış cəmi],
        DATESBETWEEN ( 'Calendar'[Date], EOMONTH ( d, -1 ) + 1, EOMONTH ( d, 0 ) ),
        REMOVEFILTERS ( 'Calendar' )
    )
```

```dax
Ötən aya görə satış =
VAR d = CALCULATE ( MAX ( 'Append'[Tarix] ) )
RETURN
    CALCULATE (
        [Satış cəmi],
        DATESBETWEEN ( 'Calendar'[Date], EOMONTH ( d, -2 ) + 1, EOMONTH ( d, -1 ) ),
        REMOVEFILTERS ( 'Calendar' )
    )
```

```dax
Ötən aya görə artım % =
DIVIDE ( [Cari ay satışı] - [Ötən aya görə satış], [Ötən aya görə satış] )
```

## Hesablanan sütunlar (Calendar)

```dax
ay    = FORMAT ( 'Calendar'[Date], "mmmm", "az" )   -- ay adına görə Ayno ilə sıralanır
İL    = FORMAT ( 'Calendar'[Date], "yyyy" )
Ayno  = MONTH ( 'Calendar'[Date] )
```
