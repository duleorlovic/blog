each time you want to remove some cards, next page will miss some items since all subsequent cards are moved down when we remove some...

solution is to use padding, so next items include moved cards...

page(X).per(Y).padding(Z).to_sql  gives limit Y, offset (X-1)*Y+Z. Offset can not be negative so padding needs to be Z >= -(X-1)*Y which is always satisfied since we have only than amount of pages.

racing conditions could occur if someone is removing cards and scrolling in the same time, so best way is to remove cards in one request, and as response increment number_of_hidden_items and than call scroll event.
