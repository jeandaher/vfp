============================================================
applications: NFP_Full_Page_App.app-meta.xml
<?xml version="1.0" encoding="UTF-8"?>
<CustomApplication xmlns="http://soap.sforce.com/2006/04/metadata">
    <label>NFP Full Page App</label>
    <formFactors>Large</formFactors>
    <navType>Standard</navType>
    <uiType>Lightning</uiType>
</CustomApplication>
============================================================
pages: NFP_Page_One.page
<apex:page showHeader="false" sidebar="false" standardStylesheets="false" docType="html-5.0">
    <html>
    <head>
        <style>
            body {
                font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
                margin: 0;
                padding: 2rem;
                background: #f8fafc;
                color: #1e293b;
            }
            .page-card {
                max-width: 800px;
                margin: 0 auto;
                background: #ffffff;
                border-radius: 8px;
                box-shadow: 0 1px 3px rgba(0,0,0,0.1);
                padding: 2rem;
            }
            h1 { font-size: 1.5rem; color: #1a1a2e; margin-top: 0; }
            p { color: #64748b; line-height: 1.6; }
            .badge {
                display: inline-block;
                padding: 0.25rem 0.75rem;
                background: #e0e7ff;
                color: #4338ca;
                border-radius: 20px;
                font-size: 0.8rem;
                font-weight: 500;
            }
        </style>
    </head>
    <body>
        <div class="page-card">
            <span class="badge">Visualforce Page 1</span>
            <h1>NFP Page One</h1>
            <p>This is a Visualforce page embedded inside the NFP Full Page App via a secure iframe container.</p>
            <p>You can add any Visualforce content here — forms, reports, Apex-driven data, etc.</p>
        </div>
    </body>
    </html>
</apex:page>
============================================================
pages: NFP_Page_One.page-mea.xml
<?xml version="1.0" encoding="UTF-8"?>
<ApexPage xmlns="http://soap.sforce.com/2006/04/metadata">
    <apiVersion>62.0</apiVersion>
    <availableInTouch>true</availableInTouch>
    <confirmationTokenRequired>false</confirmationTokenRequired>
    <label>NFP Page One</label>
</ApexPage>
============================================================
pages: NFP_Page_Two.page
<apex:page showHeader="false" sidebar="false" standardStylesheets="false" docType="html-5.0">
    <html>
    <head>
        <style>
            body {
                font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
                margin: 0;
                padding: 2rem;
                background: #f8fafc;
                color: #1e293b;
            }
            .page-card {
                max-width: 800px;
                margin: 0 auto;
                background: #ffffff;
                border-radius: 8px;
                box-shadow: 0 1px 3px rgba(0,0,0,0.1);
                padding: 2rem;
            }
            h1 { font-size: 1.5rem; color: #1a1a2e; margin-top: 0; }
            p { color: #64748b; line-height: 1.6; }
            .badge {
                display: inline-block;
                padding: 0.25rem 0.75rem;
                background: #dcfce7;
                color: #166534;
                border-radius: 20px;
                font-size: 0.8rem;
                font-weight: 500;
            }
            table {
                width: 100%;
                border-collapse: collapse;
                margin-top: 1rem;
            }
            th, td {
                text-align: left;
                padding: 0.75rem;
                border-bottom: 1px solid #e2e8f0;
            }
            th { background: #f1f5f9; font-size: 0.8rem; color: #475569; }
        </style>
    </head>
    <body>
        <div class="page-card">
            <span class="badge">Visualforce Page 2</span>
            <h1>NFP Page Two</h1>
            <p>This is the second Visualforce page with sample data.</p>
            <table>
                <thead>
                    <tr>
                        <th>Name</th>
                        <th>Status</th>
                        <th>Category</th>
                    </tr>
                </thead>
                <tbody>
                    <tr><td>Item Alpha</td><td>Active</td><td>Finance</td></tr>
                    <tr><td>Item Beta</td><td>Pending</td><td>HR</td></tr>
                    <tr><td>Item Gamma</td><td>Closed</td><td>IT</td></tr>
                </tbody>
            </table>
        </div>
    </body>
    </html>
</apex:page>

============================================================
pages: NFP_Page_Two.page-mea.xml
<?xml version="1.0" encoding="UTF-8"?>
<ApexPage xmlns="http://soap.sforce.com/2006/04/metadata">
    <apiVersion>62.0</apiVersion>
    <availableInTouch>true</availableInTouch>
    <confirmationTokenRequired>false</confirmationTokenRequired>
    <label>NFP Page Two</label>
</ApexPage>
===========================================================
lwc:nfpDataTable.html
<template>
    <div class="datatable-wrapper">
        <!-- Toolbar -->
        <div class="datatable-toolbar">
            <div class="toolbar-left">
                <template if:true={title}>
                    <h2 class="datatable-title">{title}</h2>
                </template>
                <span class="result-count">{resultCountLabel}</span>
            </div>
            <div class="toolbar-right">
                <template if:true={showSearch}>
                    <div class="search-box">
                        <lightning-icon icon-name="utility:search" size="xx-small" class="search-icon"></lightning-icon>
                        <input
                            type="text"
                            class="search-input"
                            placeholder="Search..."
                            value={searchInputValue}
                            oninput={handleSearchInput}
                        />
                        <template if:true={searchInputValue}>
                            <button class="search-clear" onclick={handleClearSearch}>
                                <lightning-icon icon-name="utility:close" size="xx-small"></lightning-icon>
                            </button>
                        </template>
                    </div>
                </template>
            </div>
        </div>

        <!-- Table -->
        <div class="datatable-scroll">
            <table class="datatable" role="grid">
                <thead>
                    <tr>
                        <template if:true={showRowNumbers}>
                            <th class="col-row-number" scope="col">#</th>
                        </template>
                        <template for:each={processedColumns} for:item="col">
                            <th
                                key={col.fieldName}
                                class={col.headerClass}
                                style={col.widthStyle}
                                scope="col"
                            >
                                <template if:true={col.sortable}>
                                    <button
                                        class="sort-btn"
                                        data-field={col.fieldName}
                                        onclick={handleSort}
                                    >
                                        <span>{col.label}</span>
                                        <template if:true={col.isSorted}>
                                            <span class="sort-indicator">{col.sortIcon}</span>
                                        </template>
                                    </button>
                                </template>
                                <template if:false={col.sortable}>
                                    <span>{col.label}</span>
                                </template>
                            </th>
                        </template>
                    </tr>
                </thead>
                <tbody>
                    <template if:true={hasData}>
                        <template for:each={displayRows} for:item="row">
                            <tr key={row._key}>
                                <template if:true={showRowNumbers}>
                                    <td class="col-row-number">{row._rowNum}</td>
                                </template>
                                <template for:each={row._cells} for:item="cell">
                                    <td key={cell._key} class={cell.cellClass}>{cell.formattedValue}</td>
                                </template>
                            </tr>
                        </template>
                    </template>
                    <template if:false={hasData}>
                        <tr>
                            <td class="empty-message" colspan={colSpan}>
                                <template if:true={searchTerm}>
                                    No results found for "{searchTerm}"
                                </template>
                                <template if:false={searchTerm}>
                                    No data available
                                </template>
                            </td>
                        </tr>
                    </template>
                </tbody>
            </table>
        </div>

        <!-- Pagination -->
        <template if:true={showPagination}>
            <div class="datatable-pagination">
                <div class="pagination-info">
                    Showing {startIndex}-{endIndex} of {totalFilteredCount}
                </div>
                <div class="pagination-controls">
                    <button
                        class="page-btn"
                        disabled={isFirstPage}
                        onclick={handleFirstPage}
                        title="First page"
                    >&laquo;</button>
                    <button
                        class="page-btn"
                        disabled={isFirstPage}
                        onclick={handlePrevPage}
                        title="Previous page"
                    >&lsaquo;</button>
                    <template for:each={pageButtons} for:item="pg">
                        <button
                            key={pg.num}
                            class={pg.btnClass}
                            data-page={pg.num}
                            onclick={handleGoToPage}
                        >{pg.label}</button>
                    </template>
                    <button
                        class="page-btn"
                        disabled={isLastPage}
                        onclick={handleNextPage}
                        title="Next page"
                    >&rsaquo;</button>
                    <button
                        class="page-btn"
                        disabled={isLastPage}
                        onclick={handleLastPage}
                        title="Last page"
                    >&raquo;</button>
                </div>
            </div>
        </template>
    </div>
</template>

===========================================================
lwc:nfpDataTable.css
:host {
    display: block;
    width: 100%;
    height: 100%;
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
}

.datatable-wrapper {
    display: flex;
    flex-direction: column;
    height: 100%;
    background: #ffffff;
    border-radius: 8px;
    box-shadow: 0 1px 3px rgba(0, 0, 0, 0.08);
    overflow: hidden;
}

/* ========================
   TOOLBAR
   ======================== */
.datatable-toolbar {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 0.75rem 1rem;
    border-bottom: 1px solid #e5e7eb;
    flex-shrink: 0;
    gap: 1rem;
}

.toolbar-left {
    display: flex;
    align-items: center;
    gap: 1rem;
}

.datatable-title {
    font-size: 1rem;
    font-weight: 600;
    color: #1f2937;
    margin: 0;
}

.result-count {
    font-size: 0.8rem;
    color: #9ca3af;
}

.toolbar-right {
    display: flex;
    align-items: center;
}

.search-box {
    display: flex;
    align-items: center;
    background: #f9fafb;
    border: 1px solid #e5e7eb;
    border-radius: 6px;
    padding: 0 0.6rem;
    height: 32px;
    width: 240px;
    transition: border-color 0.15s;
}

.search-box:focus-within {
    border-color: #6366f1;
    box-shadow: 0 0 0 2px rgba(99, 102, 241, 0.15);
}

.search-icon {
    margin-right: 0.4rem;
    flex-shrink: 0;
}

.search-input {
    flex: 1;
    border: none;
    outline: none;
    background: transparent;
    font-size: 0.8rem;
    color: #374151;
    height: 100%;
}

.search-input::placeholder {
    color: #9ca3af;
}

.search-clear {
    background: none;
    border: none;
    cursor: pointer;
    padding: 2px;
    display: flex;
    align-items: center;
    color: #9ca3af;
}

.search-clear:hover {
    color: #374151;
}

/* ========================
   TABLE SCROLL CONTAINER
   ======================== */
.datatable-scroll {
    flex: 1 1 0%;
    overflow: auto;
    min-height: 0;
    /* GPU acceleration for smooth scrolling */
    will-change: scroll-position;
    -webkit-overflow-scrolling: touch;
}

/* ========================
   TABLE
   ======================== */
.datatable {
    width: 100%;
    border-collapse: separate;
    border-spacing: 0;
    table-layout: fixed;
}

/* ---- Header ---- */
.datatable thead {
    position: sticky;
    top: 0;
    z-index: 10;
}

.datatable thead tr {
    background: #f8fafc;
}

.col-header {
    padding: 0.6rem 0.75rem;
    font-size: 0.7rem;
    font-weight: 700;
    text-transform: uppercase;
    letter-spacing: 0.04em;
    color: #64748b;
    border-bottom: 2px solid #e2e8f0;
    white-space: nowrap;
    user-select: none;
    background: #f8fafc;
}

.col-header.sortable {
    cursor: pointer;
}

.col-header.sortable:hover {
    color: #1e293b;
    background: #f1f5f9;
}

.col-left { text-align: left; }
.col-center { text-align: center; }
.col-right { text-align: right; }

.col-row-number {
    width: 50px;
    min-width: 50px;
    text-align: center;
    padding: 0.6rem 0.5rem;
    font-size: 0.7rem;
    color: #9ca3af;
    background: #f8fafc;
    border-bottom: 2px solid #e2e8f0;
    font-weight: 700;
    text-transform: uppercase;
    letter-spacing: 0.04em;
}

tbody .col-row-number {
    font-weight: 400;
    font-size: 0.75rem;
    border-bottom: 1px solid #f3f4f6;
    background: transparent;
}

.sort-btn {
    display: inline-flex;
    align-items: center;
    gap: 0.3rem;
    border: none;
    background: transparent;
    cursor: pointer;
    font: inherit;
    color: inherit;
    font-size: 0.7rem;
    font-weight: 700;
    text-transform: uppercase;
    letter-spacing: 0.04em;
    padding: 0;
}

.sort-indicator {
    font-size: 0.6rem;
    color: #4f46e5;
}

/* ---- Body rows ---- */
.datatable tbody tr {
    transition: background 0.1s;
    /* Contain layout for paint performance */
    contain: layout style;
}

.datatable tbody tr:nth-child(even) {
    background: #fafbfc;
}

.datatable tbody tr:hover {
    background: #eef2ff;
}

.cell {
    padding: 0.55rem 0.75rem;
    font-size: 0.8rem;
    color: #374151;
    border-bottom: 1px solid #f3f4f6;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
}

.cell-left { text-align: left; }
.cell-center { text-align: center; }
.cell-right { text-align: right; }

/* Empty state */
.empty-message {
    text-align: center;
    padding: 3rem 1rem;
    color: #9ca3af;
    font-size: 0.9rem;
}

/* ========================
   PAGINATION
   ======================== */
.datatable-pagination {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 0.6rem 1rem;
    border-top: 1px solid #e5e7eb;
    flex-shrink: 0;
    background: #fafbfc;
}

.pagination-info {
    font-size: 0.75rem;
    color: #6b7280;
}

.pagination-controls {
    display: flex;
    align-items: center;
    gap: 2px;
}

.page-btn {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    min-width: 32px;
    height: 32px;
    padding: 0 0.4rem;
    border: 1px solid #e5e7eb;
    border-radius: 4px;
    background: #ffffff;
    color: #374151;
    font-size: 0.75rem;
    cursor: pointer;
    transition: all 0.1s;
}

.page-btn:hover:not(:disabled):not(.active):not(.ellipsis) {
    background: #f3f4f6;
    border-color: #d1d5db;
}

.page-btn.active {
    background: #1a1a2e;
    border-color: #1a1a2e;
    color: #ffffff;
}

.page-btn:disabled {
    opacity: 0.4;
    cursor: default;
}

.page-btn.ellipsis {
    border: none;
    background: transparent;
    cursor: default;
}
===========================================================
lwc:nfpDataTable.js
import { LightningElement, api } from 'lwc';

// Cached Intl formatters — created once, reused across renders
const NUMBER_FMT = new Intl.NumberFormat(undefined, {
    minimumFractionDigits: 0,
    maximumFractionDigits: 2
});
const PERCENT_FMT = new Intl.NumberFormat(undefined, {
    style: 'percent',
    minimumFractionDigits: 1,
    maximumFractionDigits: 1
});
const DATE_FMT = new Intl.DateTimeFormat(undefined, {
    year: 'numeric',
    month: 'short',
    day: 'numeric'
});

const CURRENCY_CACHE = {};
function getCurrencyFormatter(code) {
    if (!CURRENCY_CACHE[code]) {
        CURRENCY_CACHE[code] = new Intl.NumberFormat(undefined, {
            style: 'currency',
            currency: code,
            minimumFractionDigits: 0,
            maximumFractionDigits: 2
        });
    }
    return CURRENCY_CACHE[code];
}

const MAX_PAGE_BUTTONS = 7;
const DEBOUNCE_MS = 300;

export default class NfpDataTable extends LightningElement {

    // --- Public API ---
    @api title = '';
    @api showSearch = false;
    @api showRowNumbers = false;

    @api
    get columns() { return this._columns; }
    set columns(value) {
        this._columns = value || [];
        this._searchableFields = null; // invalidate cache
    }

    @api
    get data() { return this._data; }
    set data(value) {
        this._data = value || [];
        this._filteredCache = null;
        this._sortedCache = null;
        this.currentPage = 1;
    }

    @api
    get pageSize() { return this._pageSize; }
    set pageSize(value) {
        this._pageSize = Math.max(1, parseInt(value, 10) || 50);
        this.currentPage = 1;
    }

    // --- Private state ---
    _columns = [];
    _data = [];
    _pageSize = 50;
    currentPage = 1;
    sortField = '';
    sortDirection = ''; // 'asc' | 'desc'
    searchTerm = '';
    searchInputValue = '';
    _debounceTimer;
    _filteredCache = null;
    _sortedCache = null;
    _lastFilterKey = '';
    _lastSortKey = '';
    _searchableFields = null;

    disconnectedCallback() {
        clearTimeout(this._debounceTimer);
    }

    // =====================
    // COMPUTED PIPELINE
    // =====================

    get searchableFields() {
        if (!this._searchableFields) {
            this._searchableFields = this._columns
                .filter(c => c.type === 'text' || !c.type)
                .map(c => c.fieldName);
        }
        return this._searchableFields;
    }

    /**
     * Step 1: Filter data by search term
     * Cached — only recalculates when data or searchTerm changes
     */
    get filteredData() {
        const key = `${this._data.length}:${this.searchTerm}`;
        if (this._lastFilterKey === key && this._filteredCache) {
            return this._filteredCache;
        }

        let result;
        if (!this.searchTerm) {
            result = this._data;
        } else {
            const term = this.searchTerm.toLowerCase();
            const fields = this.searchableFields;
            result = this._data.filter(row => {
                for (let i = 0; i < fields.length; i++) {
                    const val = row[fields[i]];
                    if (val != null && String(val).toLowerCase().includes(term)) {
                        return true;
                    }
                }
                return false;
            });
        }

        this._filteredCache = result;
        this._lastFilterKey = key;
        this._sortedCache = null; // invalidate sort cache
        return result;
    }

    /**
     * Step 2: Sort filtered data
     * Cached — only recalculates when filteredData, sortField, or sortDirection changes
     */
    get sortedData() {
        const filtered = this.filteredData;
        const key = `${this._lastFilterKey}:${this.sortField}:${this.sortDirection}`;
        if (this._lastSortKey === key && this._sortedCache) {
            return this._sortedCache;
        }

        let result;
        if (!this.sortField || !this.sortDirection) {
            result = filtered;
        } else {
            const field = this.sortField;
            const dir = this.sortDirection === 'asc' ? 1 : -1;
            const colConfig = this._columns.find(c => c.fieldName === field);
            const type = colConfig ? colConfig.type : 'text';

            result = [...filtered].sort((a, b) => {
                const va = a[field];
                const vb = b[field];

                if (va == null && vb == null) return 0;
                if (va == null) return dir;
                if (vb == null) return -dir;

                switch (type) {
                    case 'number':
                    case 'currency':
                    case 'percent':
                        return (Number(va) - Number(vb)) * dir;
                    case 'date':
                        return (new Date(va) - new Date(vb)) * dir;
                    case 'boolean':
                        return ((va === vb) ? 0 : va ? -1 : 1) * dir;
                    default:
                        return String(va).localeCompare(String(vb)) * dir;
                }
            });
        }

        this._sortedCache = result;
        this._lastSortKey = key;
        return result;
    }

    /**
     * Step 3: Paginate sorted data + pre-format cells
     * NOT cached — always computed (it's cheap: just a slice + format)
     */
    get displayRows() {
        const sorted = this.sortedData;
        const start = (this.currentPage - 1) * this._pageSize;
        const page = sorted.slice(start, start + this._pageSize);
        const cols = this._columns;

        return page.map((row, idx) => ({
            _key: row.id || `row-${start + idx}`,
            _rowNum: start + idx + 1,
            _cells: cols.map(col => ({
                _key: `${row.id || start + idx}-${col.fieldName}`,
                formattedValue: this._formatValue(row[col.fieldName], col),
                cellClass: `cell cell-${col.align || 'left'}`
            }))
        }));
    }

    // =====================
    // COLUMN PROCESSING
    // =====================

    get processedColumns() {
        return this._columns.map(col => ({
            ...col,
            headerClass: `col-header col-${col.align || 'left'}${col.sortable ? ' sortable' : ''}`,
            widthStyle: col.width ? `width:${col.width};min-width:${col.width}` : '',
            isSorted: this.sortField === col.fieldName,
            sortIcon: this.sortField === col.fieldName
                ? (this.sortDirection === 'asc' ? '\u25B2' : '\u25BC')
                : ''
        }));
    }

    // =====================
    // PAGINATION GETTERS
    // =====================

    get totalFilteredCount() { return this.filteredData.length; }
    get totalPages() { return Math.max(1, Math.ceil(this.totalFilteredCount / this._pageSize)); }
    get startIndex() { return this.totalFilteredCount === 0 ? 0 : (this.currentPage - 1) * this._pageSize + 1; }
    get endIndex() { return Math.min(this.currentPage * this._pageSize, this.totalFilteredCount); }
    get isFirstPage() { return this.currentPage <= 1; }
    get isLastPage() { return this.currentPage >= this.totalPages; }
    get hasData() { return this.totalFilteredCount > 0; }
    get showPagination() { return this.totalFilteredCount > this._pageSize; }
    get colSpan() { return this._columns.length + (this.showRowNumbers ? 1 : 0); }

    get resultCountLabel() {
        const total = this._data.length;
        const filtered = this.totalFilteredCount;
        if (this.searchTerm && filtered !== total) {
            return `${filtered.toLocaleString()} of ${total.toLocaleString()} results`;
        }
        return `${total.toLocaleString()} results`;
    }

    get pageButtons() {
        const total = this.totalPages;
        const current = this.currentPage;
        const buttons = [];

        if (total <= MAX_PAGE_BUTTONS) {
            for (let i = 1; i <= total; i++) {
                buttons.push(this._makePageBtn(i, current));
            }
        } else {
            buttons.push(this._makePageBtn(1, current));

            let start = Math.max(2, current - 1);
            let end = Math.min(total - 1, current + 1);

            if (current <= 3) {
                end = Math.min(5, total - 1);
            } else if (current >= total - 2) {
                start = Math.max(2, total - 4);
            }

            if (start > 2) {
                buttons.push({ num: 'ellipsis-start', label: '...', btnClass: 'page-btn ellipsis' });
            }
            for (let i = start; i <= end; i++) {
                buttons.push(this._makePageBtn(i, current));
            }
            if (end < total - 1) {
                buttons.push({ num: 'ellipsis-end', label: '...', btnClass: 'page-btn ellipsis' });
            }

            buttons.push(this._makePageBtn(total, current));
        }
        return buttons;
    }

    _makePageBtn(num, current) {
        return {
            num,
            label: String(num),
            btnClass: num === current ? 'page-btn active' : 'page-btn'
        };
    }

    // =====================
    // VALUE FORMATTING
    // =====================

    _formatValue(value, col) {
        if (value == null || value === '') return '';

        switch (col.type) {
            case 'number':
                return NUMBER_FMT.format(Number(value));
            case 'currency':
                return getCurrencyFormatter(col.currencyCode || 'USD').format(Number(value));
            case 'percent':
                return PERCENT_FMT.format(Number(value));
            case 'date': {
                const d = new Date(value);
                return isNaN(d.getTime()) ? String(value) : DATE_FMT.format(d);
            }
            case 'boolean':
                return value ? '\u2713' : '\u2717';
            default:
                return String(value);
        }
    }

    // =====================
    // EVENT HANDLERS
    // =====================

    handleSort(event) {
        const field = event.currentTarget.dataset.field;
        if (this.sortField === field) {
            this.sortDirection = this.sortDirection === 'asc' ? 'desc' : 'asc';
        } else {
            this.sortField = field;
            this.sortDirection = 'asc';
        }
        this._sortedCache = null;
        this.currentPage = 1;
    }

    handleSearchInput(event) {
        this.searchInputValue = event.target.value;
        clearTimeout(this._debounceTimer);
        this._debounceTimer = setTimeout(() => {
            this.searchTerm = this.searchInputValue;
            this._filteredCache = null;
            this._sortedCache = null;
            this.currentPage = 1;
        }, DEBOUNCE_MS);
    }

    handleClearSearch() {
        this.searchInputValue = '';
        this.searchTerm = '';
        this._filteredCache = null;
        this._sortedCache = null;
        this.currentPage = 1;
        clearTimeout(this._debounceTimer);
    }

    handleFirstPage() { this.currentPage = 1; }
    handlePrevPage() { if (this.currentPage > 1) this.currentPage--; }
    handleNextPage() { if (this.currentPage < this.totalPages) this.currentPage++; }
    handleLastPage() { this.currentPage = this.totalPages; }

    handleGoToPage(event) {
        const page = parseInt(event.currentTarget.dataset.page, 10);
        if (!isNaN(page)) {
            this.currentPage = page;
        }
    }
}

===========================================================
lwc:nfpDataTable.js-meta.xml
<?xml version="1.0" encoding="UTF-8"?>
<LightningComponentBundle xmlns="http://soap.sforce.com/2006/04/metadata">
    <apiVersion>62.0</apiVersion>
    <isExposed>false</isExposed>
    <masterLabel>NFP Data Table</masterLabel>
    <description>High-performance read-only data table with sorting, pagination, and search. Optimized for 1000+ rows.</description>
</LightningComponentBundle>

===========================================================
lwc:nfpFilterPanel.html
<template>
    <aside class={panelClass}>
        <!-- Toggle button -->
        <button class="panel-toggle" onclick={handleTogglePanel} title={toggleTitle}>
            <lightning-icon
                icon-name={toggleIcon}
                size="x-small"
            ></lightning-icon>
        </button>

        <!-- Panel content (visible when expanded) -->
        <template if:true={isPanelOpen}>
            <div class="panel-header">
                <span class="panel-title">Filters</span>
            </div>

            <div class="panel-body">
                <template for:each={sections} for:item="section">
                    <div key={section.id} class="accordion-section">
                        <button
                            class="accordion-header"
                            data-section-id={section.id}
                            onclick={handleToggleSection}
                        >
                            <span class="accordion-title">{section.label}</span>
                            <lightning-icon
                                icon-name={section.iconName}
                                size="xx-small"
                                class="accordion-chevron"
                            ></lightning-icon>
                        </button>

                        <template if:true={section.isOpen}>
                            <div class="accordion-body">
                                <template if:true={section.isCheckbox}>
                                    <template for:each={section.options} for:item="option">
                                        <label key={option.value} class="filter-option">
                                            <input
                                                type="checkbox"
                                                data-section-id={section.id}
                                                data-value={option.value}
                                                checked={option.selected}
                                                onchange={handleFilterChange}
                                            />
                                            <span class="filter-label">{option.label}</span>
                                            <template if:true={option.count}>
                                                <span class="filter-count">({option.count})</span>
                                            </template>
                                        </label>
                                    </template>
                                </template>

                                <template if:false={section.isCheckbox}>
                                    <template for:each={section.options} for:item="option">
                                        <button
                                            key={option.value}
                                            class={option.buttonClass}
                                            data-section-id={section.id}
                                            data-value={option.value}
                                            onclick={handleFilterChange}
                                        >
                                            {option.label}
                                        </button>
                                    </template>
                                </template>
                            </div>
                        </template>
                    </div>
                </template>
            </div>

            <!-- Clear all button -->
            <template if:true={hasActiveFilters}>
                <div class="panel-footer">
                    <button class="clear-all-btn" onclick={handleClearAll}>
                        Clear All Filters
                    </button>
                </div>
            </template>
        </template>
    </aside>
</template>

===========================================================
lwc:nfpFilterPanel.css
:host {
    display: block;
    height: 100%;
}

/* Panel states */
.filter-panel {
    height: 100%;
    background: #ffffff;
    border-right: 1px solid #e5e7eb;
    display: flex;
    flex-direction: column;
    transition: width 0.25s ease;
    position: relative;
}

.filter-panel.open {
    width: 280px;
    min-width: 280px;
}

.filter-panel.closed {
    width: 48px;
    min-width: 48px;
}

/* Toggle button */
.panel-toggle {
    display: flex;
    align-items: center;
    justify-content: center;
    width: 100%;
    height: 44px;
    flex-shrink: 0;
    border: none;
    background: transparent;
    cursor: pointer;
    color: #6b7280;
    border-bottom: 1px solid #e5e7eb;
    transition: background 0.15s;
}

.panel-toggle:hover {
    background: #f3f4f6;
    color: #1f2937;
}

/* Panel header */
.panel-header {
    padding: 0.75rem 1rem;
    border-bottom: 1px solid #e5e7eb;
    flex-shrink: 0;
}

.panel-title {
    font-size: 0.8rem;
    font-weight: 700;
    text-transform: uppercase;
    letter-spacing: 0.05em;
    color: #6b7280;
}

/* Panel body */
.panel-body {
    flex: 1 1 0%;
    overflow-y: auto;
    min-height: 0;
    padding: 0.5rem 0;
}

/* Accordion section */
.accordion-section {
    border-bottom: 1px solid #f3f4f6;
}

.accordion-header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    width: 100%;
    padding: 0.75rem 1rem;
    border: none;
    background: transparent;
    cursor: pointer;
    transition: background 0.15s;
}

.accordion-header:hover {
    background: #f9fafb;
}

.accordion-title {
    font-size: 0.85rem;
    font-weight: 600;
    color: #374151;
}

.accordion-chevron {
    transition: transform 0.2s;
}

/* Accordion body */
.accordion-body {
    padding: 0.25rem 1rem 0.75rem;
}

/* Checkbox filter option */
.filter-option {
    display: flex;
    align-items: center;
    gap: 0.5rem;
    padding: 0.35rem 0;
    cursor: pointer;
    font-size: 0.8rem;
    color: #4b5563;
    transition: color 0.15s;
}

.filter-option:hover {
    color: #1f2937;
}

.filter-option input[type='checkbox'] {
    width: 16px;
    height: 16px;
    accent-color: #4f46e5;
    cursor: pointer;
}

.filter-label {
    flex: 1;
}

.filter-count {
    color: #9ca3af;
    font-size: 0.75rem;
}

/* Button filter option */
.filter-btn {
    display: inline-block;
    padding: 0.3rem 0.75rem;
    margin: 0.2rem;
    border: 1px solid #d1d5db;
    border-radius: 20px;
    background: #ffffff;
    font-size: 0.75rem;
    color: #4b5563;
    cursor: pointer;
    transition: all 0.15s;
}

.filter-btn:hover {
    border-color: #4f46e5;
    color: #4f46e5;
}

.filter-btn.active {
    background: #4f46e5;
    border-color: #4f46e5;
    color: #ffffff;
}

/* Panel footer */
.panel-footer {
    padding: 0.75rem 1rem;
    border-top: 1px solid #e5e7eb;
    flex-shrink: 0;
}

.clear-all-btn {
    width: 100%;
    padding: 0.5rem;
    border: 1px solid #d1d5db;
    border-radius: 6px;
    background: transparent;
    color: #6b7280;
    font-size: 0.8rem;
    font-weight: 500;
    cursor: pointer;
    transition: all 0.15s;
}

.clear-all-btn:hover {
    background: #1a1a2e;
    border-color: #1a1a2e;
    color: #ffffff;
}

===========================================================
lwc:nfpFilterPanel.js
import { LightningElement, api, track } from 'lwc';

export default class NfpFilterPanel extends LightningElement {
    @api
    get filterConfig() {
        return this._filterConfig;
    }
    set filterConfig(value) {
        this._filterConfig = value;
        this._buildSections();
    }

    @track sections = [];
    @track _activeFilters = {};
    isPanelOpen = true;
    _filterConfig = [];

    get panelClass() {
        return this.isPanelOpen ? 'filter-panel open' : 'filter-panel closed';
    }

    get toggleIcon() {
        return this.isPanelOpen ? 'utility:chevronleft' : 'utility:filterList';
    }

    get toggleTitle() {
        return this.isPanelOpen ? 'Collapse filters' : 'Expand filters';
    }

    get hasActiveFilters() {
        return Object.values(this._activeFilters).some(
            (values) => values && values.length > 0
        );
    }

    _buildSections() {
        if (!this._filterConfig || !Array.isArray(this._filterConfig)) {
            return;
        }
        this.sections = this._filterConfig.map((config) => ({
            id: config.id,
            label: config.label,
            isOpen: config.defaultOpen !== false,
            isCheckbox: config.type !== 'button',
            iconName: config.defaultOpen !== false
                ? 'utility:chevrondown'
                : 'utility:chevronright',
            options: (config.options || []).map((opt) => ({
                ...opt,
                selected: false,
                buttonClass: 'filter-btn'
            }))
        }));
    }

    handleTogglePanel() {
        this.isPanelOpen = !this.isPanelOpen;
    }

    handleToggleSection(event) {
        const sectionId = event.currentTarget.dataset.sectionId;
        this.sections = this.sections.map((section) => {
            if (section.id === sectionId) {
                const isOpen = !section.isOpen;
                return {
                    ...section,
                    isOpen,
                    iconName: isOpen
                        ? 'utility:chevrondown'
                        : 'utility:chevronright'
                };
            }
            return section;
        });
    }

    handleFilterChange(event) {
        const sectionId =
            event.currentTarget.dataset.sectionId ||
            event.target.dataset.sectionId;
        const value =
            event.currentTarget.dataset.value || event.target.dataset.value;

        const current = this._activeFilters[sectionId]
            ? [...this._activeFilters[sectionId]]
            : [];

        const idx = current.indexOf(value);
        if (idx > -1) {
            current.splice(idx, 1);
        } else {
            current.push(value);
        }

        this._activeFilters = { ...this._activeFilters, [sectionId]: current };

        // Update visual state
        const activeValues = this._activeFilters[sectionId] || [];
        this.sections = this.sections.map((section) => {
            if (section.id === sectionId) {
                return {
                    ...section,
                    options: section.options.map((opt) => ({
                        ...opt,
                        selected: activeValues.includes(opt.value),
                        buttonClass: activeValues.includes(opt.value)
                            ? 'filter-btn active'
                            : 'filter-btn'
                    }))
                };
            }
            return section;
        });

        this._dispatchFilterEvent();
    }

    handleClearAll() {
        this._activeFilters = {};
        this.sections = this.sections.map((section) => ({
            ...section,
            options: section.options.map((opt) => ({
                ...opt,
                selected: false,
                buttonClass: 'filter-btn'
            }))
        }));
        this._dispatchFilterEvent();
    }

    _dispatchFilterEvent() {
        this.dispatchEvent(
            new CustomEvent('filterchange', {
                detail: { filters: { ...this._activeFilters } },
                bubbles: true,
                composed: true
            })
        );
    }
}

===========================================================
lwc:nfpFilterPanel.js-meta.xml
<?xml version="1.0" encoding="UTF-8"?>
<LightningComponentBundle xmlns="http://soap.sforce.com/2006/04/metadata">
    <apiVersion>62.0</apiVersion>
    <isExposed>false</isExposed>
    <masterLabel>NFP Filter Panel</masterLabel>
    <description>Accordion-based filter panel for the NFP Full Page App sidebar.</description>
</LightningComponentBundle>


======================================================
lwc:nfpFullPageApp.html
<template>
    <div class="fullpage-app">
        <!-- Header -->
        <header class="app-header">
            <div class="header-left">
                <span class="app-title">{appTitle}</span>
            </div>
            <div class="header-center">
                <div class="search-box">
                    <lightning-icon icon-name="utility:search" size="x-small" class="search-icon"></lightning-icon>
                    <input
                        type="text"
                        class="search-input"
                        placeholder="Search..."
                        value={searchTerm}
                        onchange={handleSearchChange}
                        onkeyup={handleSearchKeyUp}
                    />
                    <template if:true={searchTerm}>
                        <button class="search-clear" onclick={handleClearSearch}>
                            <lightning-icon icon-name="utility:close" size="xx-small"></lightning-icon>
                        </button>
                    </template>
                </div>
            </div>
            <div class="header-right">
                <slot name="header-actions"></slot>
            </div>
        </header>

        <!-- Body: sidebar + content -->
        <div class="app-body">
            <!-- Filter sidebar -->
            <c-nfp-filter-panel
                filter-config={filterConfig}
                onfilterchange={handleFilterChange}
            ></c-nfp-filter-panel>

            <!-- Main content -->
            <main class="app-content">
                <template if:true={activeVfPage}>
                    <c-nfp-vf-container page-name={activeVfPage}></c-nfp-vf-container>
                </template>
                <template if:true={showCustomTable}>
                    <div class="datatable-container">
                        <c-nfp-data-table
                            columns={activeTableColumns}
                            data={activeTableData}
                            title={activeTableTitle}
                            page-size="50"
                            show-search
                            show-row-numbers
                        ></c-nfp-data-table>
                    </div>
                </template>
                <template if:true={showLwcTable}>
                    <div class="datatable-container">
                        <c-nfp-l-w-c-data-table
                            columns={activeTableColumns}
                            data={activeTableData}
                            title={activeTableTitle}
                            page-size="50"
                            show-search
                            show-row-numbers
                        ></c-nfp-l-w-c-data-table>
                    </div>
                </template>
                <template if:true={showDefaultSlot}>
                    <slot></slot>
                </template>
            </main>
        </div>

        <!-- Footer -->
        <footer class="app-footer">
            <div class="footer-left">
                <span class="footer-text">{footerText}</span>
            </div>
            <div class="footer-center">
                <button class={vfPage1Class} data-page="NFP_Page_One" onclick={handleVfPageClick}>
                    Page One
                </button>
                <button class={vfPage2Class} data-page="NFP_Page_Two" onclick={handleVfPageClick}>
                    Page Two
                </button>
                <span class="footer-sep"></span>
                <button class={tableBtn1Class} data-table="cases" onclick={handleTableClick}>
                    Cases
                </button>
                <button class={tableBtn2Class} data-table="sales" onclick={handleTableClick}>
                    Sales
                </button>
                <button class={tableLwcSalesClass} data-table="lwcSales" onclick={handleTableClick}>
                    LWC Sales
                </button>
                <template if:true={hasActiveView}>
                    <button class="footer-link" onclick={handleCloseView}>
                        <lightning-icon icon-name="utility:close" size="xx-small" class="footer-link-icon"></lightning-icon>
                        Close
                    </button>
                </template>
            </div>
            <div class="footer-right">
                <slot name="footer-actions"></slot>
            </div>
        </footer>
    </div>
</template>

======================================================
lwc:nfpFullPageApp.css
:host {
    display: block;
    margin: 0;
    padding: 0;
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, sans-serif;
}

.fullpage-app {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    display: flex;
    flex-direction: column;
    background-color: #f4f6f9;
    z-index: 9999;
}

/* Header */
.app-header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    height: 56px;
    min-height: 56px;
    flex-shrink: 0;
    padding: 0 1.5rem;
    background: linear-gradient(135deg, #1a1a2e 0%, #16213e 100%);
    color: #ffffff;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.15);
    z-index: 100;
}

.header-left {
    display: flex;
    align-items: center;
    gap: 1rem;
}

.app-title {
    font-size: 1.25rem;
    font-weight: 600;
    letter-spacing: 0.02em;
}

/* Header center — search */
.header-center {
    flex: 1;
    display: flex;
    justify-content: center;
    padding: 0 2rem;
    max-width: 600px;
}

.search-box {
    display: flex;
    align-items: center;
    width: 100%;
    background: rgba(255, 255, 255, 0.12);
    border: 1px solid rgba(255, 255, 255, 0.2);
    border-radius: 6px;
    padding: 0 0.75rem;
    height: 36px;
    transition: background 0.2s, border-color 0.2s;
}

.search-box:focus-within {
    background: rgba(255, 255, 255, 0.2);
    border-color: rgba(255, 255, 255, 0.5);
}

.search-icon {
    --sds-c-icon-color-foreground-default: rgba(255, 255, 255, 0.7);
    margin-right: 0.5rem;
    flex-shrink: 0;
}

.search-input {
    flex: 1;
    background: transparent;
    border: none;
    outline: none;
    color: #ffffff;
    font-size: 0.875rem;
    height: 100%;
}

.search-input::placeholder {
    color: rgba(255, 255, 255, 0.5);
}

.search-clear {
    background: none;
    border: none;
    cursor: pointer;
    padding: 2px;
    display: flex;
    align-items: center;
    --sds-c-icon-color-foreground-default: rgba(255, 255, 255, 0.7);
}

.search-clear:hover {
    --sds-c-icon-color-foreground-default: #ffffff;
}

.header-right {
    display: flex;
    align-items: center;
    gap: 0.75rem;
}

/* Body: sidebar + content */
.app-body {
    display: flex;
    flex: 1 1 0%;
    overflow: hidden;
    min-height: 0;
}

/* Main content area */
.app-content {
    flex: 1;
    overflow: auto;
}

/* Footer */
.app-footer {
    display: flex;
    align-items: center;
    justify-content: space-between;
    height: 40px;
    min-height: 40px;
    flex-shrink: 0;
    padding: 0 1.5rem;
    background: #1a1a2e;
    color: rgba(255, 255, 255, 0.6);
    font-size: 0.75rem;
    border-top: 1px solid rgba(255, 255, 255, 0.1);
    z-index: 100;
}

.footer-left {
    display: flex;
    align-items: center;
}

.footer-center {
    display: flex;
    align-items: center;
    gap: 0.25rem;
}

.footer-link {
    display: inline-flex;
    align-items: center;
    gap: 0.3rem;
    padding: 0.25rem 0.75rem;
    border: 1px solid rgba(255, 255, 255, 0.2);
    border-radius: 4px;
    background: transparent;
    color: rgba(255, 255, 255, 0.6);
    font-size: 0.75rem;
    cursor: pointer;
    transition: all 0.15s;
}

.footer-link:hover {
    border-color: rgba(255, 255, 255, 0.5);
    color: #ffffff;
    background: rgba(255, 255, 255, 0.1);
}

.footer-link.active {
    border-color: #818cf8;
    color: #ffffff;
    background: rgba(129, 140, 248, 0.2);
}

.footer-link-icon {
    --sds-c-icon-color-foreground-default: currentColor;
}

.footer-sep {
    width: 1px;
    height: 20px;
    background: rgba(255, 255, 255, 0.2);
    margin: 0 0.4rem;
}

.footer-right {
    display: flex;
    align-items: center;
    gap: 0.75rem;
}

/* Data table container — fill available space */
.datatable-container {
    height: 100%;
    padding: 0.75rem;
}

======================================================
lwc:nfpFullPageApp.js
import { LightningElement, api } from 'lwc';
import {
    CASES_COLUMNS, generateCasesData,
    SALES_COLUMNS, generateSalesData
} from './nfpMockData';

const TABLE_CONFIGS = {
    cases: {
        title: 'Cases',
        columns: CASES_COLUMNS,
        generator: generateCasesData,
        component: 'custom'
    },
    sales: {
        title: 'Sales Pipeline',
        columns: SALES_COLUMNS,
        generator: generateSalesData,
        component: 'custom'
    },
    lwcSales: {
        title: 'Sales Pipeline (LWC)',
        columns: SALES_COLUMNS,
        generator: generateSalesData,
        component: 'lwc'
    }
};

const STYLE_ID = 'nfp-fullpage-override';

export default class NfpFullPageApp extends LightningElement {
    @api appTitle = 'My App';
    @api footerText = '\u00A9 2025 My App. All rights reserved.';

    searchTerm = '';
    activeVfPage = '';
    activeTable = ''; // 'cases' | 'sales' | ''
    activeTableColumns = [];
    activeTableData = [];
    activeTableTitle = '';
    _styleElement;

    // Default demo filter config — override via @api filterConfig
    @api filterConfig = [
        {
            id: 'status',
            label: 'Status',
            defaultOpen: true,
            type: 'checkbox',
            options: [
                { value: 'new', label: 'New' },
                { value: 'in_progress', label: 'In Progress' },
                { value: 'completed', label: 'Completed' },
                { value: 'closed', label: 'Closed' }
            ]
        },
        {
            id: 'priority',
            label: 'Priority',
            defaultOpen: false,
            type: 'checkbox',
            options: [
                { value: 'high', label: 'High' },
                { value: 'medium', label: 'Medium' },
                { value: 'low', label: 'Low' }
            ]
        },
        {
            id: 'category',
            label: 'Category',
            defaultOpen: false,
            type: 'button',
            options: [
                { value: 'all', label: 'All' },
                { value: 'finance', label: 'Finance' },
                { value: 'hr', label: 'HR' },
                { value: 'it', label: 'IT' },
                { value: 'legal', label: 'Legal' }
            ]
        }
    ];

    connectedCallback() {
        this._injectOverrideStyles();
    }

    disconnectedCallback() {
        this._removeOverrideStyles();
    }

    handleSearchChange(event) {
        this.searchTerm = event.target.value;
    }

    handleSearchKeyUp(event) {
        if (event.key === 'Enter') {
            this.dispatchEvent(new CustomEvent('search', {
                detail: { searchTerm: this.searchTerm }
            }));
        }
    }

    handleFilterChange(event) {
        this.dispatchEvent(new CustomEvent('filterchange', {
            detail: event.detail
        }));
    }

    // VF page footer links
    get vfPage1Class() {
        return this.activeVfPage === 'NFP_Page_One'
            ? 'footer-link active'
            : 'footer-link';
    }

    get vfPage2Class() {
        return this.activeVfPage === 'NFP_Page_Two'
            ? 'footer-link active'
            : 'footer-link';
    }

    handleVfPageClick(event) {
        const page = event.currentTarget.dataset.page;
        if (this.activeVfPage === page) {
            this.activeVfPage = '';
        } else {
            this.activeVfPage = page;
            this.activeTable = '';
        }
    }

    // Data table footer links
    get tableBtn1Class() {
        return this.activeTable === 'cases' ? 'footer-link active' : 'footer-link';
    }

    get tableBtn2Class() {
        return this.activeTable === 'sales' ? 'footer-link active' : 'footer-link';
    }

    get tableLwcSalesClass() {
        return this.activeTable === 'lwcSales' ? 'footer-link active' : 'footer-link';
    }

    get activeTableComponent() {
        if (!this.activeTable) return '';
        const config = TABLE_CONFIGS[this.activeTable];
        return config ? config.component : '';
    }

    get showCustomTable() {
        return !this.activeVfPage && this.activeTableComponent === 'custom';
    }

    get showLwcTable() {
        return !this.activeVfPage && this.activeTableComponent === 'lwc';
    }

    get showDefaultSlot() {
        return !this.activeVfPage && !this.activeTable;
    }

    get hasActiveView() {
        return !!this.activeVfPage || !!this.activeTable;
    }

    handleTableClick(event) {
        const tableKey = event.currentTarget.dataset.table;
        if (this.activeTable === tableKey) {
            this.activeTable = '';
            return;
        }
        this.activeVfPage = '';
        const config = TABLE_CONFIGS[tableKey];
        if (config) {
            this.activeTableTitle = config.title;
            this.activeTableColumns = config.columns;
            this.activeTableData = config.generator();
            this.activeTable = tableKey;
        }
    }

    handleCloseView() {
        this.activeVfPage = '';
        this.activeTable = '';
    }

    handleClearSearch() {
        this.searchTerm = '';
        this.dispatchEvent(new CustomEvent('search', {
            detail: { searchTerm: '' }
        }));
    }

    _injectOverrideStyles() {
        if (document.getElementById(STYLE_ID)) {
            return;
        }

        this._styleElement = document.createElement('style');
        this._styleElement.id = STYLE_ID;
        this._styleElement.textContent = `
            /* Hide Salesforce global header */
            .slds-global-header_container,
            .communitiesHeader,
            header.slds-global-header {
                display: none !important;
            }

            /* Hide Salesforce navigation bar */
            .oneAppNavContainer,
            .appNavContainer,
            .tabBarContainer,
            .slds-context-bar {
                display: none !important;
            }

            /* Prevent Salesforce body from scrolling behind our fixed app */
            body {
                overflow: hidden !important;
            }
        `;
        document.head.appendChild(this._styleElement);
    }

    _removeOverrideStyles() {
        const existing = document.getElementById(STYLE_ID);
        if (existing) {
            existing.remove();
        }
        this._styleElement = null;
    }
}

======================================================
lwc:nfpFullPageApp.js-meta.xml
<?xml version="1.0" encoding="UTF-8"?>
<LightningComponentBundle xmlns="http://soap.sforce.com/2006/04/metadata">
    <apiVersion>62.0</apiVersion>
    <isExposed>true</isExposed>
    <masterLabel>NFP Full Page App</masterLabel>
    <description>Full-page container that hides Salesforce header and navigation for a standalone web app experience.</description>
    <targets>
        <target>lightning__AppPage</target>
        <target>lightning__Tab</target>
    </targets>
    <targetConfigs>
        <targetConfig targets="lightning__AppPage">
            <property name="appTitle" type="String" label="App Title" default="My App" description="Title displayed in the app header"/>
            <property name="footerText" type="String" label="Footer Text" default="© 2025 My App. All rights reserved." description="Text displayed in the footer"/>
        </targetConfig>
    </targetConfigs>
</LightningComponentBundle>

======================================================
lwc:nfpFullPageApp file nfpMockData.js 

// Mock data generators for demo tables

const STATUSES = ['Active', 'Pending', 'Closed', 'In Review', 'Approved'];
const CATEGORIES = ['Finance', 'HR', 'IT', 'Legal', 'Operations', 'Marketing'];
const REGIONS = ['North', 'South', 'East', 'West', 'Central'];
const FIRST_NAMES = ['Alice', 'Bob', 'Clara', 'David', 'Eva', 'Frank', 'Grace', 'Hugo', 'Iris', 'Jack', 'Karen', 'Leo', 'Mia', 'Noah', 'Olivia', 'Paul', 'Quinn', 'Rose', 'Sam', 'Tina'];
const LAST_NAMES = ['Martin', 'Dubois', 'Bernard', 'Thomas', 'Petit', 'Robert', 'Richard', 'Durand', 'Moreau', 'Laurent', 'Simon', 'Michel', 'Garcia', 'David', 'Bertrand'];
const PRODUCTS = ['Alpha Suite', 'Beta Platform', 'Gamma Cloud', 'Delta Analytics', 'Epsilon AI', 'Zeta Security', 'Eta Connect', 'Theta CRM'];

function randomItem(arr) {
    return arr[Math.floor(Math.random() * arr.length)];
}

function randomDate(startYear, endYear) {
    const start = new Date(startYear, 0, 1).getTime();
    const end = new Date(endYear, 11, 31).getTime();
    return new Date(start + Math.random() * (end - start)).toISOString().split('T')[0];
}

// ==========================================
// TABLE 1 — Cases (1200 rows)
// ==========================================
export const CASES_COLUMNS = [
    { fieldName: 'caseNumber', label: 'Case #', type: 'text', sortable: true, width: '100px' },
    { fieldName: 'subject', label: 'Subject', type: 'text', sortable: true, width: '250px' },
    { fieldName: 'status', label: 'Status', type: 'text', sortable: true, width: '100px' },
    { fieldName: 'priority', label: 'Priority', type: 'text', sortable: true, width: '90px' },
    { fieldName: 'category', label: 'Category', type: 'text', sortable: true, width: '110px' },
    { fieldName: 'amount', label: 'Amount', type: 'currency', sortable: true, align: 'right', currencyCode: 'EUR' },
    { fieldName: 'createdDate', label: 'Created', type: 'date', sortable: true, width: '120px' },
    { fieldName: 'resolved', label: 'Resolved', type: 'boolean', align: 'center', width: '80px' }
];

export function generateCasesData(count = 1200) {
    const data = [];
    const priorities = ['High', 'Medium', 'Low', 'Critical'];
    const subjects = [
        'System access request', 'Budget approval needed', 'License renewal',
        'Data migration issue', 'Security audit finding', 'Compliance review',
        'Vendor onboarding', 'Employee offboarding', 'Infrastructure upgrade',
        'Performance issue reported', 'Integration failure', 'Report discrepancy'
    ];

    for (let i = 0; i < count; i++) {
        const status = randomItem(STATUSES);
        data.push({
            id: `case-${i}`,
            caseNumber: `CS-${String(10000 + i).padStart(6, '0')}`,
            subject: `${randomItem(subjects)} - ${randomItem(REGIONS)}`,
            status,
            priority: randomItem(priorities),
            category: randomItem(CATEGORIES),
            amount: Math.round(Math.random() * 50000 * 100) / 100,
            createdDate: randomDate(2023, 2025),
            resolved: status === 'Closed' || status === 'Approved'
        });
    }
    return data;
}

// ==========================================
// TABLE 2 — Sales Pipeline (800 rows)
// ==========================================
export const SALES_COLUMNS = [
    { fieldName: 'dealName', label: 'Deal', type: 'text', sortable: true, width: '220px' },
    { fieldName: 'contact', label: 'Contact', type: 'text', sortable: true, width: '160px' },
    { fieldName: 'product', label: 'Product', type: 'text', sortable: true, width: '140px' },
    { fieldName: 'region', label: 'Region', type: 'text', sortable: true, width: '90px' },
    { fieldName: 'value', label: 'Value', type: 'currency', sortable: true, align: 'right', currencyCode: 'EUR' },
    { fieldName: 'probability', label: 'Prob.', type: 'percent', sortable: true, align: 'right', width: '80px' },
    { fieldName: 'closeDate', label: 'Close Date', type: 'date', sortable: true, width: '120px' },
    { fieldName: 'won', label: 'Won', type: 'boolean', align: 'center', width: '60px' }
];

export function generateSalesData(count = 800) {
    const data = [];
    const companies = [
        'Acme Corp', 'Global Tech', 'Nova Industries', 'Summit Group',
        'Pacific Systems', 'Atlas Partners', 'Zenith Corp', 'Apex Digital',
        'Titan Solutions', 'Vertex Inc', 'Horizon Labs', 'Pinnacle Ltd'
    ];
    const stages = ['Prospecting', 'Qualification', 'Proposal', 'Negotiation', 'Closed Won', 'Closed Lost'];

    for (let i = 0; i < count; i++) {
        const prob = Math.round(Math.random() * 100) / 100;
        const stage = randomItem(stages);
        data.push({
            id: `deal-${i}`,
            dealName: `${randomItem(companies)} - ${randomItem(PRODUCTS)}`,
            contact: `${randomItem(FIRST_NAMES)} ${randomItem(LAST_NAMES)}`,
            product: randomItem(PRODUCTS),
            region: randomItem(REGIONS),
            value: Math.round(Math.random() * 200000 * 100) / 100,
            probability: prob,
            closeDate: randomDate(2025, 2026),
            won: stage === 'Closed Won'
        });
    }
    return data;
}


=======================================================
lwc:nfpLWCDataTable.html
<template>
    <div class="lwc-table-wrapper">
        <!-- Toolbar -->
        <div class="table-toolbar">
            <template if:true={title}>
                <h2 class="table-title">{title}</h2>
            </template>
            <div class="toolbar-spacer"></div>
            <template if:true={showSearch}>
                <div class="table-search">
                    <lightning-input
                        type="search"
                        label="Search"
                        variant="label-hidden"
                        placeholder="Search…"
                        value={searchTerm}
                        onchange={handleSearchInput}
                    ></lightning-input>
                </div>
            </template>
        </div>

        <!-- Lightning Data Table -->
        <div class="table-container">
            <lightning-datatable
                key-field="id"
                data={paginatedData}
                columns={_ltColumns}
                sorted-by={sortedBy}
                sorted-direction={sortDirection}
                onsort={handleSort}
                show-row-number-column={showRowNumbers}
                row-number-offset={startIndex}
                hide-checkbox-column
            ></lightning-datatable>
        </div>

        <!-- Pagination -->
        <template if:true={showPagination}>
            <div class="pagination">
                <span class="pagination-info">{paginationInfo}</span>
                <div class="pagination-buttons">
                    <button class="page-btn" disabled={isFirstPage} onclick={handleFirst}>« First</button>
                    <button class="page-btn" disabled={isFirstPage} onclick={handlePrev}>‹ Prev</button>
                    <template for:each={pageButtons} for:item="btn">
                        <template if:true={btn.isEllipsis}>
                            <span key={btn.key} class="page-ellipsis">…</span>
                        </template>
                        <template if:false={btn.isEllipsis}>
                            <button
                                key={btn.key}
                                class={btn.cssClass}
                                data-page={btn.num}
                                onclick={handlePageClick}
                            >{btn.label}</button>
                        </template>
                    </template>
                    <button class="page-btn" disabled={isLastPage} onclick={handleNext}>Next ›</button>
                    <button class="page-btn" disabled={isLastPage} onclick={handleLast}>Last »</button>
                </div>
            </div>
        </template>
    </div>
</template>

=======================================================
lwc:nfpLWCDataTable.css
:host {
    display: block;
    height: 100%;
}

.lwc-table-wrapper {
    display: flex;
    flex-direction: column;
    height: 100%;
    background: #ffffff;
    border-radius: 8px;
    box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
    overflow: hidden;
}

/* Toolbar */
.table-toolbar {
    display: flex;
    align-items: center;
    padding: 0.75rem 1rem;
    border-bottom: 1px solid #e5e7eb;
    flex-shrink: 0;
    gap: 1rem;
}

.table-title {
    font-size: 1rem;
    font-weight: 600;
    color: #1a1a2e;
    margin: 0;
    white-space: nowrap;
}

.toolbar-spacer {
    flex: 1;
}

.table-search {
    width: 260px;
    flex-shrink: 0;
}

/* Table container — fill remaining space */
.table-container {
    flex: 1 1 0%;
    overflow: auto;
    min-height: 0;
}

/* Pagination */
.pagination {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 0.5rem 1rem;
    border-top: 1px solid #e5e7eb;
    background: #f9fafb;
    flex-shrink: 0;
}

.pagination-info {
    font-size: 0.8rem;
    color: #6b7280;
}

.pagination-buttons {
    display: flex;
    align-items: center;
    gap: 4px;
}

.page-btn {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    min-width: 32px;
    height: 32px;
    padding: 0 8px;
    border: 1px solid #d1d5db;
    border-radius: 4px;
    background: #ffffff;
    color: #374151;
    font-size: 0.8rem;
    cursor: pointer;
    transition: all 0.15s;
}

.page-btn:hover:not(:disabled):not(.active) {
    background: #f3f4f6;
    border-color: #9ca3af;
}

.page-btn.active {
    background: #1a1a2e;
    color: #ffffff;
    border-color: #1a1a2e;
}

.page-btn:disabled {
    opacity: 0.4;
    cursor: not-allowed;
}

.page-ellipsis {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    min-width: 32px;
    height: 32px;
    color: #6b7280;
    font-size: 0.8rem;
}

=======================================================
lwc:nfpLWCDataTable.js
import { LightningElement, api } from 'lwc';

export default class NfpLWCDataTable extends LightningElement {
    @api title = '';
    @api showSearch = false;
    @api showRowNumbers = false;
    @api pageSize = 50;

    _columns = [];
    _data = [];
    _ltColumns = [];

    searchTerm = '';
    _searchTimer;
    _appliedSearch = '';

    sortedBy = '';
    sortDirection = 'asc';
    currentPage = 1;

    // Caches
    _filteredData = [];
    _lastFilterKey = '';
    _sortedData = [];
    _lastSortKey = '';

    @api
    get columns() { return this._columns; }
    set columns(value) {
        this._columns = value || [];
        this._ltColumns = this._convertColumns(this._columns);
        this._invalidateCache();
    }

    @api
    get data() { return this._data; }
    set data(value) {
        this._data = value || [];
        this._invalidateCache();
    }

    _invalidateCache() {
        this._lastFilterKey = '';
        this._lastSortKey = '';
        this.currentPage = 1;
    }

    /* ------------------------------------------------
     * Convert custom column format → lightning-datatable
     * ------------------------------------------------ */
    _convertColumns(cols) {
        return cols.map(col => {
            const ltCol = {
                label: col.label,
                fieldName: col.fieldName,
                type: col.type,
                sortable: !!col.sortable
            };

            if (col.width) {
                const px = parseInt(col.width, 10);
                if (!isNaN(px)) ltCol.initialWidth = px;
            }

            if (col.type === 'currency' && col.currencyCode) {
                ltCol.typeAttributes = { currencyCode: col.currencyCode };
            }
            if (col.type === 'date') {
                ltCol.typeAttributes = {
                    year: 'numeric',
                    month: 'short',
                    day: '2-digit'
                };
            }
            if (col.type === 'percent') {
                ltCol.typeAttributes = {
                    minimumFractionDigits: 0,
                    maximumFractionDigits: 0
                };
            }

            if (col.align) {
                ltCol.cellAttributes = { alignment: col.align };
            }

            return ltCol;
        });
    }

    /* ------------------------------------------------
     * Search (debounced 300ms)
     * ------------------------------------------------ */
    handleSearchInput(event) {
        this.searchTerm = event.detail.value;
        clearTimeout(this._searchTimer);
        this._searchTimer = setTimeout(() => {
            this._appliedSearch = this.searchTerm.trim().toLowerCase();
            this._lastFilterKey = '';
            this._lastSortKey = '';
            this.currentPage = 1;
        }, 300);
    }

    get hasSearch() { return !!this.searchTerm; }

    /* ------------------------------------------------
     * Filtered data
     * ------------------------------------------------ */
    get filteredData() {
        const key = `${this._data.length}|${this._appliedSearch}`;
        if (key === this._lastFilterKey) return this._filteredData;

        if (!this._appliedSearch) {
            this._filteredData = this._data;
        } else {
            const term = this._appliedSearch;
            const textFields = this._columns
                .filter(c => c.type === 'text')
                .map(c => c.fieldName);
            this._filteredData = this._data.filter(row =>
                textFields.some(f => {
                    const val = row[f];
                    return val && String(val).toLowerCase().includes(term);
                })
            );
        }
        this._lastFilterKey = key;
        return this._filteredData;
    }

    /* ------------------------------------------------
     * Sorted data
     * ------------------------------------------------ */
    get sortedData() {
        const key = `${this._lastFilterKey}|${this.sortedBy}|${this.sortDirection}`;
        if (key === this._lastSortKey) return this._sortedData;

        const data = [...this.filteredData];
        if (this.sortedBy) {
            const col = this._columns.find(c => c.fieldName === this.sortedBy);
            const dir = this.sortDirection === 'asc' ? 1 : -1;
            data.sort((a, b) => {
                const va = a[this.sortedBy];
                const vb = b[this.sortedBy];
                if (va == null && vb == null) return 0;
                if (va == null) return dir;
                if (vb == null) return -dir;

                if (col && (col.type === 'number' || col.type === 'currency' || col.type === 'percent')) {
                    return (va - vb) * dir;
                }
                if (col && col.type === 'date') {
                    return (new Date(va) - new Date(vb)) * dir;
                }
                if (col && col.type === 'boolean') {
                    return ((va === vb) ? 0 : va ? -1 : 1) * dir;
                }
                return String(va).localeCompare(String(vb)) * dir;
            });
        }
        this._sortedData = data;
        this._lastSortKey = key;
        return this._sortedData;
    }

    /* ------------------------------------------------
     * Pagination
     * ------------------------------------------------ */
    get totalRecords() { return this.sortedData.length; }
    get totalPages() { return Math.max(1, Math.ceil(this.totalRecords / this.pageSize)); }
    get startIndex() { return (this.currentPage - 1) * this.pageSize; }
    get endIndex() { return Math.min(this.startIndex + this.pageSize, this.totalRecords); }
    get paginatedData() { return this.sortedData.slice(this.startIndex, this.endIndex); }

    get showPagination() { return this.totalRecords > this.pageSize; }
    get isFirstPage() { return this.currentPage === 1; }
    get isLastPage() { return this.currentPage === this.totalPages; }

    get paginationInfo() {
        const start = this.totalRecords === 0 ? 0 : this.startIndex + 1;
        return `Showing ${start.toLocaleString()}\u2013${this.endIndex.toLocaleString()} of ${this.totalRecords.toLocaleString()}`;
    }

    get pageButtons() {
        const MAX = 7;
        const total = this.totalPages;
        const current = this.currentPage;
        const buttons = [];

        if (total <= MAX) {
            for (let i = 1; i <= total; i++) {
                buttons.push({
                    key: `p${i}`, num: i, label: String(i),
                    isCurrent: i === current, isEllipsis: false,
                    cssClass: i === current ? 'page-btn active' : 'page-btn'
                });
            }
        } else {
            buttons.push({
                key: 'p1', num: 1, label: '1',
                isCurrent: current === 1, isEllipsis: false,
                cssClass: current === 1 ? 'page-btn active' : 'page-btn'
            });

            let start = Math.max(2, current - 1);
            let end = Math.min(total - 1, current + 1);
            if (current <= 3) { start = 2; end = 5; }
            if (current >= total - 2) { start = total - 4; end = total - 1; }

            if (start > 2) {
                buttons.push({ key: 'e-start', num: null, label: '…', isCurrent: false, isEllipsis: true });
            }
            for (let i = start; i <= end; i++) {
                buttons.push({
                    key: `p${i}`, num: i, label: String(i),
                    isCurrent: i === current, isEllipsis: false,
                    cssClass: i === current ? 'page-btn active' : 'page-btn'
                });
            }
            if (end < total - 1) {
                buttons.push({ key: 'e-end', num: null, label: '…', isCurrent: false, isEllipsis: true });
            }

            buttons.push({
                key: `p${total}`, num: total, label: String(total),
                isCurrent: current === total, isEllipsis: false,
                cssClass: current === total ? 'page-btn active' : 'page-btn'
            });
        }
        return buttons;
    }

    /* ------------------------------------------------
     * Event handlers
     * ------------------------------------------------ */
    handleSort(event) {
        this.sortedBy = event.detail.fieldName;
        this.sortDirection = event.detail.sortDirection;
        this._lastSortKey = '';
        this.currentPage = 1;
    }

    handleFirst() { this.currentPage = 1; }
    handlePrev() { if (this.currentPage > 1) this.currentPage--; }
    handleNext() { if (this.currentPage < this.totalPages) this.currentPage++; }
    handleLast() { this.currentPage = this.totalPages; }
    handlePageClick(event) {
        const page = Number(event.target.dataset.page);
        if (page) this.currentPage = page;
    }
}

=======================================================
lwc:nfpLWCDataTable.js-meta.xml
<?xml version="1.0" encoding="UTF-8"?>
<LightningComponentBundle xmlns="http://soap.sforce.com/2006/04/metadata">
    <apiVersion>62.0</apiVersion>
    <isExposed>false</isExposed>
    <masterLabel>NFP LWC Data Table</masterLabel>
    <description>Wrapper around lightning-datatable with search, sorting, and pagination.</description>
</LightningComponentBundle>

=======================================================
lwc:nfpVfContainer.html
<template>
    <div class="vf-container">
        <template if:true={iframeSrc}>
            <iframe
                src={iframeSrc}
                title={pageTitle}
                class="vf-iframe"
                sandbox="allow-scripts allow-same-origin allow-forms allow-popups allow-popups-to-escape-sandbox"
                referrerpolicy="strict-origin-when-cross-origin"
            ></iframe>
        </template>
        <template if:false={iframeSrc}>
            <div class="vf-placeholder">
                <lightning-icon icon-name="utility:page" size="large"></lightning-icon>
                <p>No Visualforce page selected.</p>
            </div>
        </template>
    </div>
</template>

=======================================================
lwc:nfpVfContainer.css
:host {
    display: block;
    width: 100%;
    height: 100%;
}

.vf-container {
    width: 100%;
    height: 100%;
    position: relative;
}

.vf-iframe {
    width: 100%;
    height: 100%;
    border: none;
    display: block;
}

.vf-placeholder {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    height: 100%;
    color: #9ca3af;
    gap: 1rem;
}

.vf-placeholder p {
    font-size: 0.9rem;
    margin: 0;
}

=======================================================
lwc:nfpVfContainer.js
import { LightningElement, api } from 'lwc';

// Whitelist of allowed VF page names — prevents arbitrary URL injection
const ALLOWED_PAGES = new Set([
    'NFP_Page_One',
    'NFP_Page_Two'
]);

export default class NfpVfContainer extends LightningElement {
    _pageName = '';
    _iframeSrc = '';

    @api
    get pageName() {
        return this._pageName;
    }
    set pageName(value) {
        this._pageName = value;
        this._resolvePageUrl();
    }

    get iframeSrc() {
        return this._iframeSrc;
    }

    get pageTitle() {
        return this._pageName
            ? `Visualforce: ${this._pageName}`
            : 'Visualforce Page';
    }

    _resolvePageUrl() {
        const name = this._pageName ? this._pageName.trim() : '';

        if (!name) {
            this._iframeSrc = '';
            return;
        }

        // Security: only allow whitelisted page names
        if (!ALLOWED_PAGES.has(name)) {
            console.error(
                `[nfpVfContainer] Page "${name}" is not in the allowed list.`
            );
            this._iframeSrc = '';
            return;
        }

        // Sanitize: ensure name contains only valid characters (alphanumeric + underscore)
        if (!/^[a-zA-Z][a-zA-Z0-9_]*$/.test(name)) {
            console.error(
                `[nfpVfContainer] Invalid page name: "${name}"`
            );
            this._iframeSrc = '';
            return;
        }

        // Build safe relative URL to the VF page
        this._iframeSrc = `/apex/${encodeURIComponent(name)}`;
    }
}


=======================================================
lwc:nfpVfContainer.js-meta.xml
<?xml version="1.0" encoding="UTF-8"?>
<LightningComponentBundle xmlns="http://soap.sforce.com/2006/04/metadata">
    <apiVersion>62.0</apiVersion>
    <isExposed>false</isExposed>
    <masterLabel>NFP Visualforce Container</masterLabel>
    <description>Secure iframe container for embedding Visualforce pages inside LWC.</description>
</LightningComponentBundle>






