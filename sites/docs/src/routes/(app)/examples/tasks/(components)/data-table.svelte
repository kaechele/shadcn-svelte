<script lang="ts" generics="TData, TValue">
	import {
		type ColumnDef,
		type ColumnFiltersState,
		type PaginationState,
		type RowSelectionState,
		type SortingState,
		type VisibilityState,
		getCoreRowModel,
		getFacetedRowModel,
		getFacetedUniqueValues,
		getFilteredRowModel,
		getPaginationRowModel,
		getSortedRowModel,
	} from "@tanstack/table-core";
	import DataTableToolbar from "./data-table-toolbar.svelte";
	import DataTablePagination from "./data-table-pagination.svelte";
	import {
		createSvelteTable,
		createTableState,
	} from "$lib/registry/new-york/ui/data-table/data-table.svelte.js";
	import FlexRender from "$lib/registry/new-york/ui/data-table/flex-render.svelte";
	import * as Table from "$lib/registry/new-york/ui/table/index.js";

	let {
		columns,
		data,
		defaultPageSize = 10,
	}: { columns: ColumnDef<TData, TValue>[]; data: TData[]; defaultPageSize: number } = $props();

	const [getRowSelection, setRowSelection] = createTableState<RowSelectionState>({});
	const [getColumnVisibility, setColumnVisibility] = createTableState<VisibilityState>({});
	const [getColumnFilters, setColumnFilters] = createTableState<ColumnFiltersState>([]);
	const [getSorting, setSorting] = createTableState<SortingState>([]);
	const [getPagination, setPagination] = createTableState<PaginationState>({
		pageIndex: 0,
		pageSize: defaultPageSize,
	});

	const table = createSvelteTable({
		get data() {
			return data;
		},
		state: {
			get sorting() {
				return getSorting();
			},
			get columnVisibility() {
				return getColumnVisibility();
			},
			get rowSelection() {
				return getRowSelection();
			},
			get columnFilters() {
				return getColumnFilters();
			},
			get pagination() {
				return getPagination();
			},
		},
		columns,
		enableRowSelection: true,
		onRowSelectionChange: setRowSelection,
		onSortingChange: setSorting,
		onColumnFiltersChange: setColumnFilters,
		onColumnVisibilityChange: setColumnVisibility,
		onPaginationChange: setPagination,
		getCoreRowModel: getCoreRowModel(),
		getFilteredRowModel: getFilteredRowModel(),
		getPaginationRowModel: getPaginationRowModel(),
		getSortedRowModel: getSortedRowModel(),
		getFacetedRowModel: getFacetedRowModel(),
		getFacetedUniqueValues: getFacetedUniqueValues(),
	});
</script>

<div class="space-y-4">
	<DataTableToolbar {table} />
	<div class="rounded-md border">
		<Table.Root>
			<Table.Header>
				{#each table.getHeaderGroups() as headerGroup (headerGroup.id)}
					<Table.Row>
						{#each headerGroup.headers as header (header.id)}
							<Table.Head colspan={header.colSpan}>
								{#if !header.isPlaceholder}
									<FlexRender
										content={header.column.columnDef.header}
										context={header.getContext()}
									/>
								{/if}
							</Table.Head>
						{/each}
					</Table.Row>
				{/each}
			</Table.Header>
			<Table.Body>
				{#each table.getRowModel().rows as row (row.id)}
					<Table.Row data-state={row.getIsSelected() && "selected"}>
						{#each row.getVisibleCells() as cell (cell.id)}
							<Table.Cell>
								<FlexRender
									content={cell.column.columnDef.cell}
									context={cell.getContext()}
								/>
							</Table.Cell>
						{/each}
					</Table.Row>
				{:else}
					<Table.Row>
						<Table.Cell colspan={columns.length} class="h-24 text-center">
							No results.
						</Table.Cell>
					</Table.Row>
				{/each}
			</Table.Body>
		</Table.Root>
	</div>
	<DataTablePagination {table} />
</div>
