# Product Slug-Based Routing Implementation Guide

## Implementation Summary

### ✅ Completed Changes

#### Backend
1. **Product Model** (`server/models/Product.js`)
   - Already has `slug` field with unique index
   - Pre-save hook auto-generates slug from product title using slugify
   - Slug is ensured to be unique on creation/update

2. **API Endpoints** (`server/routes/products.js`)
   - New endpoint: `GET /api/products/slug/:slug` - Fetch product by slug
   - Existing endpoint: `GET /api/products/:idOrSlug` - Supports both ID and slug (backward compatible)
   - All create/update endpoints handle slug generation and uniqueness

3. **Migration Script** (`server/scripts/backfill-slugs.js`)
   - Backfills existing products that don't have slugs
   - Ensures uniqueness by appending -1, -2, etc. if needed
   - Run manually: `node server/scripts/backfill-slugs.js`

#### Frontend
1. **Routing** (`src/App.tsx`)
   - New route: `/products/:slug` → ProductDetail component
   - Old route: `/product/:id` → ProductRedirect component (for backward compatibility)
   - 301 redirect support via ProductRedirect component

2. **Product Detail Page** (`src/pages/ProductDetail.tsx`)
   - Uses `slug` parameter instead of `id`
   - Fetches product via `/api/products/slug/:slug`
   - Canonical link set to slug URL for SEO
   - Meta tags include OG:url, description, title, image

3. **Redirect Component** (`src/pages/ProductRedirect.tsx`)
   - Handles old `/product/:id` URLs
   - Fetches product by ID to get slug
   - Redirects to `/products/:slug`

4. **Product Links** (Updated across the app)
   - `src/components/ProductCard.tsx` - Accepts slug prop, generates `/products/:slug` links
   - `src/pages/Products.tsx` - Uses slug in product links
   - `src/pages/Shop.tsx` - Passes slug to ProductCard
   - `src/pages/Index.tsx` - Uses slug in product links
   - `src/pages/Wishlist.tsx` - Uses slug in product links
   - `src/components/RelatedProducts.tsx` - Uses slug in product links

5. **SEO Improvements**
   - Canonical link tag pointing to slug URL
   - Open Graph meta tags with proper URLs
   - Twitter card image
   - Dynamic page title with product name and price
   - Meta description with product info

## Testing Instructions

### Prerequisites
1. Products in database must have slugs
2. Run migration: `node server/scripts/backfill-slugs.js`

### Test Cases

#### 1. Access product by slug (new URL format)
- **URL**: `https://uni10.in/products/bark-t-shirt`
- **Expected**: Product loads correctly
- **Verify**: Product detail page displays product info, images, prices

#### 2. Access product by old ID format
- **URL**: `https://uni10.in/product/69180bc9df7fc95d6dcd953f` (example ID)
- **Expected**: Redirects to `/products/bark-t-shirt`
- **Verify**: URL changes to slug-based, product loads

#### 3. Product cards in Shop page
- **Action**: Visit `/shop`
- **Expected**: Product cards link to `/products/:slug` URLs
- **Verify**: Click product card, verify URL and product detail loads

#### 4. Product cards in Index page
- **Action**: Visit home page `/`
- **Expected**: New arrivals carousel links use slug URLs
- **Verify**: Click product, verify slug-based URL

#### 5. Related products section
- **Action**: Open any product detail
- **Expected**: Related products section links use slug URLs
- **Verify**: Click related product, verify slug-based URL

#### 6. Wishlist page
- **Action**: Add products to wishlist, visit `/wishlist`
- **Expected**: Wishlist product links use slug URLs
- **Verify**: Click product, verify slug-based URL

#### 7. Search functionality
- **Action**: Search for a product
- **Expected**: Search results link to slug-based URLs
- **Verify**: Click result, product loads with slug URL

#### 8. SEO verification
- **URL**: `https://uni10.in/products/bark-t-shirt`
- **Expected**: Check page source or DevTools
- **Verify**:
  - `<link rel="canonical" href="...products/bark-t-shirt">`
  - `<meta property="og:url" content="...products/bark-t-shirt">`
  - `<title>` contains product name
  - `<meta name="description">` contains product info

#### 9. Slug uniqueness
- **Action**: Create/update products with similar names
- **Expected**: Slugs are unique
- **Verify**: 
  - "Bark T-Shirt" → `bark-t-shirt`
  - "Bark T-Shirt" (duplicate) → `bark-t-shirt-1`

#### 10. Empty stock / inactive products
- **Action**: Create product, set inactive or out of stock
- **Expected**: Slug still generated correctly
- **Verify**: Can access via slug URL, shows correct status

## Migration Checklist

- [ ] Run: `node server/scripts/backfill-slugs.js`
- [ ] Verify all products have slugs in database
- [ ] Clear browser cache
- [ ] Test slug URLs in production
- [ ] Monitor error logs for redirect issues
- [ ] Test on mobile devices
- [ ] Verify social sharing (WhatsApp, Instagram) with new URLs
- [ ] Update internal links (admin area, documentation)

## Important Notes

1. **Database**: The migration script is non-destructive - it only fills in missing slugs
2. **Backward Compatibility**: Old `/product/:id` URLs still work via redirects
3. **SEO**: Canonical links ensure search engines understand the preferred URL
4. **Mobile**: All changes are fully responsive and work on mobile
5. **Performance**: Slug queries use indexes (same performance as ID queries)

## Example Slug Transformations

```
"Bark T-Shirt" → "bark-t-shirt"
"Oversized Black Printed Tee" → "oversized-black-printed-tee"
"LIMITED EDITION - Super Cool Hoodie 2024" → "limited-edition-super-cool-hoodie-2024"
"Men's Cargo Pants (30-40)" → "mens-cargo-pants-30-40"
```

## Rollback Instructions

If needed to revert:

1. Keep old routes in `src/App.tsx` for `/product/:id`
2. All old product data is preserved
3. Database slugs can be removed: `db.products.updateMany({}, { $unset: { slug: "" } })`
4. No data loss - only URL format changes

## Support & Debugging

### Issue: Product not found at slug URL
- **Cause**: Slug might not be generated yet
- **Solution**: Run migration script: `node server/scripts/backfill-slugs.js`

### Issue: Redirect loop
- **Cause**: ProductRedirect component infinite loop
- **Solution**: Check if product exists in database, verify slug generation

### Issue: Old links not working
- **Cause**: ProductRedirect component not in routes
- **Solution**: Verify `/product/:id` route points to ProductRedirect component

### Issue: SEO meta tags not updating
- **Cause**: Browser cache
- **Solution**: Hard refresh (Ctrl+Shift+R), or check after page load completes
