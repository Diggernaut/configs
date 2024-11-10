---
config:
    agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/71.0.3578.98 Safari/537.36
    debug: 2
do:
# Push start URL to the pool
- link_add:
    url: https://www.olx.ua/detskiy-mir/detskaya-odezhda/dnepr/
# Iterate over the pool and load each link
- walk:
    to: links
    do:
    # Find the link to the next page
    - find:
        path: a[data-cy="page-link-next"]
        do:
        # Parse link from href attribute
        - parse:
            attr: href
        # Add it to the pool
        - link_add
    # Find a link to an ad
    - find:
        path: a.link.detailsLink
        do:
        # Parse URL from href attribute
        - parse:
            attr: href
        - variable_set:
            field: repeat
            value: "yes"
        # Load page with the ad
        - walk:
            to: value
            repeat: <%repeat%>
            do:
            - variable_clear: ok
            # Find common container for the ad
            - find:
                path: 'div#offer_active'
                do:
                - variable_set:
                    field: ok
                    value: 1
                # Create data object with name item
                - object_new: item
                # Find element with ad title
                - find:
                    path: h1
                    do:
                    # Parse text content
                    - parse
                    # Normalize parsed data, depupe and trim whitespaces
                    - space_dedupe
                    - trim
                    # Save data to the item data object field
                    - object_field_set:
                        object: item
                        field: title
                # Find element with ad description
                - find:
                    path: 'div#textContent'
                    do:
                    # Parse text content
                    - parse
                    # Normalize parsed data, depupe and trim whitespaces
                    - space_dedupe
                    - trim
                    # Save data to the item data object field
                    - object_field_set:
                        object: item
                        field: description
                # Find element with ad ID
                - find:
                    path: 'em > small'
                    do:
                    # Parse text content using the filter. Since ID consist with digits only, we will apply filter to extract only digits.
                    - parse:
                        filter: (\d+)
                    # Save data to the item data object field
                    - object_field_set:
                        object: item
                        field: ad_id
                # Find element with ad date and time
                - find:
                    path: 'em'
                    do:
                    # Remove nodes with not relevant information
                    - node_remove: a,small
                    # Parse text content
                    - parse
                    # Normalize parsed data, depupe and trim whitespaces
                    - space_dedupe
                    - trim
                    # Remove trailing comma
                    - normalize:
                        routine: replace_substring
                        args:
                            \,$: ''
                    # Save data to the item data object field
                    - object_field_set:
                        object: item
                        field: date
                # Find element with ad price
                - find:
                    path: div.price-label
                    do:
                    # Parse text content
                    - parse
                    # Normalize parsed data, depupe and trim whitespaces
                    - space_dedupe
                    - trim
                    # Save data to the item data object field
                    - object_field_set:
                        object: item
                        field: price
                # Find element with seller name
                - find:
                    path: div.offer-user__details > h4
                    do:
                    # Parse text content
                    - parse
                    # Normalize parsed data, depupe and trim whitespaces
                    - space_dedupe
                    - trim
                    # Save data to the item data object field
                    - object_field_set:
                        object: item
                        field: seller
                # Find element with address
                - find:
                    path: address > p
                    do:
                    # Parse text content
                    - parse
                    # Normalize parsed data, depupe and trim whitespaces
                    - space_dedupe
                    - trim
                    # Save data to the item data object field
                    - object_field_set:
                        object: item
                        field: address
                # Find element with image
                - find:
                    path: div#photo-gallery-opener > img
                    do:
                    # Parse content of src attribute and filter it to cut the end with size
                    - parse:
                        attr: src
                        filter: ^([^;]+)
                    # Save data to the item data object field
                    - object_field_set:
                        object: item
                        field: image
                # Let's also save ad URL to the data object
                # we will use content of static variable "url" for it
                - static_get: url
                # Save data to the item data object field
                - object_field_set:
                    object: item
                    field: url
                # Now let's get data from the table with item details
                - find:
                    path: table.details
                    do:
                    # Find all table rows which has child cell with class "value"
                    - find:
                        path: tr:haschild(td.value)
                        do:
                        # Switch to th to get field name
                        - find:
                            path: th
                            do:
                            # Parse text content
                            - parse
                            # Normalize parsed data, depupe and trim whitespaces
                            - space_dedupe
                            - trim
                            # Save content to the variable "fieldname"
                            - variable_set: fieldname
                        # Switch to td to get field data
                        - find:
                            path: td
                            do:
                            # Parse text content
                            - parse
                            # Normalize parsed data, depupe and trim whitespaces
                            - space_dedupe
                            - trim
                            # Save data to the field defined by name kept in the "fieldname" variable of the "item" object
                            - object_field_set:
                                object: item
                                field: <%fieldname%>
                # Find the script element with phonetoken (we need to lookup in whole document as currently we are in the block without this script tag)
                - find:
                    in: doc
                    path: script:contains("phoneToken")
                    do:
                    # Parse only token using regular expression
                    - parse:
                        filter: \'([^']+)\'
                    # Save value to the variable
                    - variable_set: token
                # Find the "Show phone" button
                - find:
                    path: li.link-phone
                    do:
                    # Parse ID of the ad
                    - parse:
                        attr: class
                        filter: \'id\'\:\'([^']+)\'
                    # Save value to the variable
                    - variable_set: id
                    # Do random pause from 5 to 10 sec
                    - sleep: 5:10
                    # Send request to the server
                    - walk:
                        to: https://www.olx.ua/uk/ajax/misc/contact/phone/<%id%>/?pt=<%token%>
                        headers:
                            accept: '*/*'
                            accept-language: ru-RU,ru;q=0.9,en-US;q=0.8,en;q=0.7
                            x-requested-with: XMLHttpRequest
                        do:
                        # Find element with phone number
                        - find:
                            path: body_safe > value
                            do:
                            # Parse text
                            - parse
                            # Save data to the item data object field
                            - object_field_set:
                                object: item
                                field: phone
                # Save object item to the dataset
                - object_save:
                    name: item
                - cookie_reset
            - find:
                path: body
                do:
                - variable_get: ok
                - if:
                    match: 1
                    do:
                    - variable_clear: repeat
                    else:
                    - error: Proxy is banned or page layout has been changed
                    - cookie_reset
                    - proxy_switch
    - cookie_reset
