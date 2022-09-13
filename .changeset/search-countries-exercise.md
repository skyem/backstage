---
'@backstage/plugin-search-react': minor
---

You can now pass a `query` prop to the `SearchResult` component when you want to request results from the Search API instead of consuming them from a parent Search Context.

Example:

```jsx
const SearchPage = () => {
  const query = {
    term: 'example',
  };

  return (
    <SearchResult query={query}>
      {({ results }) => (
        <List>
          {results.map(({ document }) => (
            <DefaultResultListItem key={document.location} result={document} />
          ))}
        </List>
      )}
    </SearchResult>
  );
};
```

And your search page can be composed using these two new results layout components:

```jsx
// Example rendering results as list
<SearchResult>
  {({ results }) => (
    <SearchResultListLayout
      resultItems={results}
      renderResultItem={({ type, document }) => {
        switch (type) {
          case 'custom-result-item':
            return (
              <CustomResultListItem key={document.location} result={document} />
            );
          default:
            return (
              <DefaultResultListItem
                key={document.location}
                result={document}
              />
            );
        }
      }}
    />
  )}
</SearchResult>
```

```jsx
// Example rendering results as groups
<SearchResult>
  {({ results }) => (
    <>
      <SearchResultGroupLayout
        icon={<CustomIcon />}
        title="Custom"
        link="See all custom results"
        resultItems={results.filter(
          ({ type }) => type === 'custom-result-item',
        )}
        renderResultItem={({ document }) => (
          <CustomResultListItem key={document.location} result={document} />
        )}
      />
      <SearchResultGroupLayout
        icon={<DefaultIcon />}
        title="Default"
        resultItems={results.filter(
          ({ type }) => type !== 'custom-result-item',
        )}
        renderResultItem={({ document }) => (
          <DefaultResultListItem key={document.location} result={document} />
        )}
      />
    </>
  )}
</SearchResult>
```

For those who are on a **multi-query** search page and want to have individual lists or groups of results we created two components that are syntax sugars for `SearchResult` and they also accepts an optional `query` to consume results from the search API instead of context:

_SearchResultList_

```jsx
const SearchPage = () => {
  const query = {
    term: 'example',
  };

  return (
    <SearchResultList
      query={query}
      renderResultItem={({ type, document, highlight, rank }) => {
        switch (type) {
          case 'custom':
            return (
              <CustomResultListItem
                key={document.location}
                icon={<CatalogIcon />}
                result={document}
                highlight={highlight}
                rank={rank}
              />
            );
          default:
            return (
              <DefaultResultListItem
                key={document.location}
                result={document}
              />
            );
        }
      }}
    />
  );
};
```

_SearchResultGroup_

```jsx
// Example creating a component that search and group software catalog results
import React, { useState, useCallback } from 'react';

import { MenuItem } from '@material-ui/core';

import { JsonValue } from '@backstage/types';
import { CatalogIcon } from '@backstage/core-components';
import { CatalogSearchResultListItem } from '@backstage/plugin-catalog';
import {
  SearchResultGroup,
  SearchResultGroupTextFilterField,
  SearchResultGroupSelectFilterField,
} from @backstage/plugin-search-react;
import { SearchQuery } from '@backstage/plugin-search-common';

const CatalogResultsGroup = () => {
  const [query, setQuery] = useState<Partial<SearchQuery>>({
    types: ['software-catalog'],
  });

  const filterOptions = [
    {
      label: 'Lifecycle',
      value: 'lifecycle',
    },
    {
      label: 'Owner',
      value: 'owner',
    },
  ];

  const handleFilterAdd = useCallback(
    (key: string) => () => {
      setQuery(prevQuery => {
        const { filters: prevFilters, ...rest } = prevQuery;
        const newFilters = { ...prevFilters, [key]: undefined };
        return { ...rest, filters: newFilters };
      });
    },
    [],
  );

  const handleFilterChange = useCallback(
    (key: string) => (value: JsonValue) => {
      setQuery(prevQuery => {
        const { filters: prevFilters, ...rest } = prevQuery;
        const newFilters = { ...prevFilters, [key]: value };
        return { ...rest, filters: newFilters };
      });
    },
    [],
  );

  const handleFilterDelete = useCallback(
    (key: string) => () => {
      setQuery(prevQuery => {
        const { filters: prevFilters, ...rest } = prevQuery;
        const newFilters = { ...prevFilters };
        delete newFilters[key];
        return { ...rest, filters: newFilters };
      });
    },
    [],
  );

  return (
    <SearchResultGroup
      query={query}
      icon={<CatalogIcon />}
      title="Software Catalog"
      link="See all software catalog results"
      filterOptions={filterOptions}
      renderFilterOption={({ label, value }) => (
        <MenuItem key={value} onClick={handleFilterAdd(value)}>
          {label}
        </MenuItem>
      )}
      renderFilterField={(key: string) => (
        <>
          {key === 'lifecycle' && (
            <SearchResultGroupSelectFilterField
              key={key}
              label="Lifecycle"
              value={query.filters?.lifecycle}
              onChange={handleFilterChange('lifecycle')}
              onDelete={handleFilterDelete('lifecycle')}
            >
              <MenuItem value="production">Production</MenuItem>
              <MenuItem value="experimental">Experimental</MenuItem>
            </SearchResultGroupSelectFilterField>
          )}
          {key === 'owner' && (
            <SearchResultGroupTextFilterField
              key={key}
              label="Owner"
              value={query.filters?.owner}
              onChange={handleFilterChange('owner')}
              onDelete={handleFilterDelete('owner')}
            />
          )}
        </>
      )}
      renderResultItem={({ document, highlight, rank }) => (
        <CatalogSearchResultListItem
          key={document.location}
          result={document}
          highlight={highlight}
          rank={rank}
        />
      )}
    />
  );
};
```
