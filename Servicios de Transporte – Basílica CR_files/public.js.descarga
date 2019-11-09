/**
 * Extend default number object with format function
 *
 * @param integer n: length of decimal
 * @param integer x: length of whole part
 * @param mixed   s: sections delimiter
 * @param mixed   c: decimal delimiter
 */
Number.prototype.jetFormat = function( n, x, s, c ) {
	var re = '\\d(?=(\\d{' + (x || 3) + '})+' + (n > 0 ? '\\D' : '$') + ')',
	num = this.toFixed(Math.max(0, ~~n));
	return (c ? num.replace('.', c) : num).replace(new RegExp(re, 'g'), '$&' + (s || ','));
};

( function( $ ) {

	"use strict";

	var JetSmartFilterSettings = window.JetSmartFilterSettings;
	var xhr = null;

	var JetSmartFilters = {

		currentQuery: null,
		page: false,
		controls: false,

		init: function() {

			var self = JetSmartFilters;

			$( document )
				.on( 'click.JetSmartFilters', '.apply-filters__button[data-apply-type="reload"]', self.applyFilters )
				.on( 'click.JetSmartFilters', 'button[data-apply-type="ajax-reload"]', self.applyAjaxFilters )
				.on( 'click.JetSmartFilters', 'button[data-apply-type="ajax"]', self.applyAjaxFilters )
				.on( 'click.JetSmartFilters', '.jet-active-filter', self.removeFilter )
				.on( 'change.JetSmartFilters', 'input[data-apply-type="ajax"]', self.applyAjaxFilters )
				.on( 'change.JetSmartFilters', 'select[data-apply-type="ajax"]', self.applyAjaxFilters )
				.on( 'keypress.JetSmartFilters', 'input.jet-search-filter__input', self.applySearchFilterOnEnter )

				.on( 'click.JetSmartFilters', '.jet-filters-pagination__link', self.applyPagination )

				.on( 'jet-filter-add-rating-vars', self.processRating )
				.on( 'jet-filter-add-checkboxes-vars', self.processCheckbox )
				.on( 'jet-filter-add-check-range-vars', self.processCheckbox )
				.on( 'jet-filter-add-range-vars', self.processRange )
				.on( 'jet-filter-add-date-range-vars', self.processRange )
				.on( 'jet-filter-add-select-vars', self.processSelect )
				.on( 'jet-filter-add-search-vars', self.processSearch )
				.on( 'jet-filter-add-radio-vars', self.processRadio )

				.on( 'jet-filter-remove-checkboxes-vars', self.removeCheckbox )
				.on( 'jet-filter-remove-check-range-vars', self.removeCheckbox )
				.on( 'jet-filter-remove-range-vars', self.removeRange )
				.on( 'jet-filter-remove-date-range-vars', self.removeDateRange )
				.on( 'jet-filter-remove-select-vars', self.removeSelect )
				.on( 'jet-filter-remove-search-vars', self.removeSearch )
				.on( 'jet-filter-remove-radio-vars', self.removeCheckbox )
				.on( 'jet-filter-remove-rating-vars', self.removeCheckbox )

				.on( 'click.JetSmartFilters', 'input.jet-rating-star__input', self.unselectRating)

				.on( 'jet-filter-load', self.applyLoader )
				.on( 'jet-filter-loaded', self.removeLoader )

				.on( 'jet-engine-request-calendar', self.addFiltersToCalendarRequest );

			$( window ).on( 'elementor/frontend/init', function() {

				if ( JetSmartFilterSettings.refresh_controls ) {

					$.each( JetSmartFilterSettings.refresh_provider, function( provider, instances ) {
						$.each( instances, function( index, queryID ) {
							setTimeout( function() {
								self.refreshControls( provider, queryID );
							} );
						});
					});
				}

			} );

		},

		addFiltersToCalendarRequest: function( event ) {
			window.JetEngine.currentRequest.query    = JetSmartFilters.getQuery( 'object', 'jet-engine-calendar' );
			window.JetEngine.currentRequest.provider = 'jet-engine-calendar';
		},

		providerSelector: function( providerWrap, queryID ) {

			var delimiter = '';

			if ( providerWrap.inDepth ) {
				delimiter = ' ';
			}

			return providerWrap.idPrefix + queryID + delimiter + providerWrap.selector;

		},

		applyLoader: function ( event, $scope, JetSmartFilters, provider, query, queryID ) {

			var providerWrap = JetSmartFilterSettings.selectors[ provider ],
				$provider    = null;

			if ( ! queryID ) {
				queryID = 'default';
			}

			if ( 'default' === queryID ) {
				$provider = $( providerWrap.selector );
			} else {
				$provider = $( JetSmartFilters.providerSelector( providerWrap, queryID ) );
			}

			$provider.addClass( 'jet-filters-loading' );
			$( 'div[data-content-provider="' + provider + '"][data-query-id="' + queryID + '"], button[data-apply-provider="' + provider + '"][data-query-id="' + queryID + '"]' ).addClass( 'jet-filters-loading' );

		},

		removeLoader: function ( event, $scope, JetSmartFilters, provider, query, queryID ) {

			var providerWrap = JetSmartFilterSettings.selectors[ provider ],
				$provider    = null;

			if ( ! queryID ) {
				queryID = 'default';
			}

			if ( 'default' === queryID ) {
				$provider = $( providerWrap.selector );
			} else {
				$provider = $( JetSmartFilters.providerSelector( providerWrap, queryID ) );
			}

			$provider.removeClass( 'jet-filters-loading' );
			$( 'div[data-content-provider="' + provider + '"][data-query-id="' + queryID + '"], button[data-apply-provider="' + provider + '"][data-query-id="' + queryID + '"]' ).removeClass( 'jet-filters-loading' );

			if( $scope.hasClass('jet-filters-pagination__link') && $scope.data('apply-type') ){
				$('html, body').stop().animate({ scrollTop: $provider.offset().top }, 500);
			}

		},

		removeFilter: function() {

			var $filter    = $( this ),
				$filters   = $filter.closest( '.jet-active-filters' ),
				data       = $filter.data( 'filter' ),
				provider   = $filters.data( 'apply-provider' ),
				reloadType = $filters.data( 'apply-type' );

			$( document ).trigger(
				'jet-filter-remove-' + data.type + '-vars',
				[ $filter, data, JetSmartFilters, provider ]
			);

			if ( 'ajax' !== reloadType ) {
				JetSmartFilters.requestFilter( provider );
			}

		},

		removeRange: function( event, $scope, data, JetSmartFilters, provider ) {

			var filterID  = data.id,
				$filter   = JetSmartFilters.getFilterElement( filterID ),
				$slider   = $filter.find( '.jet-range__slider' ),
				$input    = $filter.find( '.jet-range__input' ),
				$min      = $filter.find( '.jet-range__values-min' ),
				$max      = $filter.find( '.jet-range__values-max' ),
				min       = $slider.data( 'min' ),
				max       = $slider.data( 'max' ),
				applyType = $input.data( 'apply-type' );

			$slider.slider( 'values', [ min, max ] );

			$min.text( min );
			$max.text( max );

			$input.val( min + ':' + max );

			if ( 'ajax' === applyType ) {
				$input.trigger( 'change.JetSmartFilters' );
			} else if ( 'ajax-reload' === applyType ) {
				JetSmartFilters.ajaxFilters( $input );
			}

		},

		removeDateRange: function( event, $scope, data, JetSmartFilters, provider ) {

			var filterID = data.id,
				$filter  = JetSmartFilters.getFilterElement( filterID ),
				$from    = $filter.find( '.jet-date-range__from' ),
				$to      = $filter.find( '.jet-date-range__to' ),
				$input   = $filter.find( '.jet-date-range__input' ),
				$submit  = $filter.find( '.jet-date-range__submit' );

			$from.val( '' );
			$to.val( '' );
			$input.val( '' );
			$submit.trigger( 'click.JetSmartFilters' );
		},

		removeCheckbox: function( event, $scope, data, JetSmartFilters, provider ) {

			var filterID  = data.id,
				$last     = null,
				$filter   = JetSmartFilters.getFilterElement( filterID ),
				applyType = null;

			$filter.find( 'input:checked' ).each( function() {
				var $this = $( this );
				$this.removeAttr( 'checked' );
				$last = $this;
			});

			if ( $last ) {

				applyType = $last.data( 'apply-type' );

				if ( 'ajax' === applyType ) {
					$last.trigger( 'change.JetSmartFilters' );
				} else if ( 'ajax-reload' === applyType ) {
					JetSmartFilters.ajaxFilters( $last );
				}
			}

		},

		removeSelect: function( event, $scope, data, JetSmartFilters, provider ) {

			var filterID  = data.id,
				$select   = JetSmartFilters.getFilterElement( filterID, 'div[data-filter-id="' + filterID + '"] select' ),
				applyType = $select.data( 'apply-type' );

			$select.find( 'option:selected' ).removeAttr( 'selected' );

			if ( 'ajax' === applyType ) {
				$select.trigger( 'change.JetSmartFilters' );
			} else if ( 'ajax-reload' === applyType ) {
				JetSmartFilters.ajaxFilters( $select );
			}

		},

		removeSearch: function( event, $scope, data, JetSmartFilters, provider ) {

			var filterID = data.id,
				$filter  = JetSmartFilters.getFilterElement( filterID );

			$filter.find( 'input' ).val( '' );
			$filter.find( '.jet-search-filter__submit' ).trigger( 'click.JetSmartFilters' );

		},

		unselectRating : function (){

			var $this = $(this);

			if ( $this.hasClass('is-checked') ){

				$this.attr('checked', false);
				$this.removeClass('is-checked');

				var applyType = $this.data( 'apply-type' );

				if ( 'ajax' === applyType ) {
					$this.trigger( 'change.JetSmartFilters' );
				}

			} else {
				$this.siblings().removeClass('is-checked');
				$this.addClass('is-checked');
			}

		},

		getFilterElement: function( filterID, selector ) {

			if ( ! selector ) {
				selector = 'div[data-filter-id="' + filterID + '"]';
			}

			var $el = $( selector );

			if ( ! $el.length ) {

				if ( window.elementorProFrontend && window.elementorFrontend ) {

					$.each( window.elementorFrontend.documentsManager.documents, function( index, elementorDocument ) {
						if ( 'popup' === elementorDocument.$element.data( 'elementor-type' ) ) {

							var $popupEl = elementorDocument.$element.find( selector );

							if ( $popupEl.length ) {
								$el = $popupEl;
							}
						}
					});

				}
			}

			return $el;

		},

		addQueryVarSuffix: function( queryVar, $scope ) {

			var queryVarSuffix = $scope.data( 'query-var-suffix' );

			if ( queryVarSuffix ) {
				queryVar = queryVar + '|' + queryVarSuffix;
			}

			return queryVar;
		},

		processSearch: function( event, $scope, queryVar, JetSmartFilters, queryType ) {

			queryVar = JetSmartFilters.addQueryVarSuffix( queryVar, $scope );

			var val = $scope.find( 'input[type="search"]' ).val();

			if ( ! val ) {
				return;
			}

			if ( 'url' === queryType ) {

				JetSmartFilters.currentQuery = JetSmartFilters.addQueryArg(
					queryVar,
					val,
					JetSmartFilters.currentQuery,
					'replace'
				);

			} else {
				JetSmartFilters.currentQuery[ queryVar ] = val;
			}

		},

		processRadio: function( event, $scope, queryVar, JetSmartFilters, queryType ) {

			queryVar = JetSmartFilters.addQueryVarSuffix( queryVar, $scope );

			var val = $scope.find( 'input:checked' ).val();

			if ( ! val ) {
				return;
			}

			if ( 'url' === queryType ) {

				JetSmartFilters.currentQuery = JetSmartFilters.addQueryArg(
					queryVar,
					val,
					JetSmartFilters.currentQuery,
					'replace'
				);

			} else {
				JetSmartFilters.currentQuery[ queryVar ] = val;
			}

		},

		processRating: function( event, $scope, queryVar, JetSmartFilters, queryType ) {

			queryVar = JetSmartFilters.addQueryVarSuffix( queryVar, $scope );

			var val = $scope.find( 'input:checked' ).addClass('is-checked').val();

			if ( ! val ) {
				return;
			}

			if ( 'url' === queryType ) {

				JetSmartFilters.currentQuery = JetSmartFilters.addQueryArg(
					queryVar,
					val,
					JetSmartFilters.currentQuery,
					'replace'
				);

			} else {
				JetSmartFilters.currentQuery[ queryVar ] = val;
			}

		},

		processSelect: function( event, $scope, queryVar, JetSmartFilters, queryType ) {

			queryVar = JetSmartFilters.addQueryVarSuffix( queryVar, $scope );

			var val = $scope.find( 'option:selected' ).val();

			if ( ! val ) {
				return;
			}

			if ( 'url' === queryType ) {

				JetSmartFilters.currentQuery = JetSmartFilters.addQueryArg(
					queryVar,
					val,
					JetSmartFilters.currentQuery,
					'replace'
				);

			} else {

				if ( JetSmartFilters.currentQuery[ queryVar ] ) {
					JetSmartFilters.currentQuery[ queryVar + '|multi_select' ] = [ JetSmartFilters.currentQuery[ queryVar ] ];
					JetSmartFilters.currentQuery[ queryVar + '|multi_select' ].push( val );
					delete JetSmartFilters.currentQuery[ queryVar ];
				} else {
					JetSmartFilters.currentQuery[ queryVar ] = val;
				}

			}

		},

		processRange: function( event, $scope, queryVar, JetSmartFilters, queryType ) {

			queryVar = JetSmartFilters.addQueryVarSuffix( queryVar, $scope );

			var val     = $scope.find( 'input[type="hidden"]' ).val(),
				$slider = $scope.find( '.jet-range__slider' ),
				values  = val.split( ':' );

			if ( ! values[0] && ! values[1] ) {
				return;
			}

			// Prevent of adding slider defaults
			if ( $slider.length ) {

				var min = $slider.data( 'min' ),
					max = $slider.data( 'max' );

				if ( values[0] && values[0] == min && values[1] && values[1] == max ) {
					return;
				}

			}

			if ( ! val ) {
				return;
			}

			if ( 'url' === queryType ) {

				JetSmartFilters.currentQuery = JetSmartFilters.addQueryArg(
					queryVar,
					val,
					JetSmartFilters.currentQuery,
					'replace'
				);

			} else {
				JetSmartFilters.currentQuery[ queryVar ] = val;
			}

		},

		processCheckbox: function( event, $scope, queryVar, JetSmartFilters, queryType ) {

			queryVar = JetSmartFilters.addQueryVarSuffix( queryVar, $scope );

			if ( 'url' === queryType ) {
				queryVar = queryVar + '[]';
			}

			$scope.find( 'input:checked' ).each( function() {

				if ( 'url' === queryType ) {

					JetSmartFilters.currentQuery = JetSmartFilters.addQueryArg(
						queryVar,
						$( this ).val(),
						JetSmartFilters.currentQuery,
						'append'
					);

				} else {

					if ( JetSmartFilters.currentQuery[ queryVar ] ) {
						JetSmartFilters.currentQuery[ queryVar ].push( $( this ).val() );
					} else {
						JetSmartFilters.currentQuery[ queryVar ] = [ $( this ).val() ];
					}

				}

			} );

		},

		applyPagination: function() {

			var $this      = $( this ),
				reloadType = $this.data( 'apply-type' ),
				queryID = $this.data( 'query-id' ),
				provider   = $this.data( 'apply-provider' );

			JetSmartFilters.page     = $this.data( 'page' );
			JetSmartFilters.controls = $this.closest( '.jet-smart-filters-pagination' ).data( 'controls' );

			if ( 'ajax' === reloadType ) {
				JetSmartFilters.ajaxFilters( $this );
			} else {
				JetSmartFilters.requestFilter( provider, queryID );
			}


		},

		applySearchFilterOnEnter: function( e ) {

			if ( 'keypress' === e.type && 13 === e.keyCode){
				var $this    = $( this ),
					provider = $this.data( 'apply-provider' ),
					applyType = $this.data( 'apply-type' ),
					queryID  = $this.data( 'query-id' );

				if ( 'ajax-reload' === applyType ){
					JetSmartFilters.ajaxFilters( $this );
				} else {
					JetSmartFilters.requestFilter( provider, queryID );
				}
			}

		},

		applyAjaxFilters: function() {

			JetSmartFilters.ajaxFilters( $( this ) );

		},

		refreshControls: function( provider, queryID ) {

			var query  = JetSmartFilters.getQuery( 'object', provider ),
				props  = null,
				paginationType = 'ajax',
				$pager = $( '.jet-smart-filters-pagination[data-content-provider="' + provider + '"][data-query-id="' + queryID + '"]' );

			if ( xhr ) {
				xhr.abort();
			}

			if ( $pager.length ) {
				JetSmartFilters.controls = $pager.data( 'controls' );
				paginationType = $pager.data( 'apply-type' );
			}

			if ( JetSmartFilterSettings.props && JetSmartFilterSettings.props[ provider ] && JetSmartFilterSettings.props[ provider ][ queryID ] ) {
				props = JetSmartFilterSettings.props[ provider ][ queryID ];
			}

			var action = 'ajax' === paginationType ? 'jet_smart_filters_refresh_controls' : 'jet_smart_filters_refresh_controls_reload';

			xhr = JetSmartFilters.ajaxRequest(
				false,
				action,
				provider,
				query,
				props,
				queryID
			);

		},

		ajaxFilters: function( $scope ) {

			var provider = $scope.data( 'apply-provider' ),
				queryID  = $scope.data( 'query-id' ),
				props    = null,
				$pager   = $( '.jet-smart-filters-pagination[data-content-provider="' + provider + '"][data-query-id="' + queryID + '"]' ),
				query    = {};

			if ( ! queryID ) {
				queryID = 'default';
			}

			query = JetSmartFilters.getQuery( 'object', provider, queryID );

			if ( xhr ) {
				xhr.abort();
			}

			if ( $pager.length ) {
				JetSmartFilters.controls = $pager.data( 'controls' );
			}

			$( document ).trigger(
				'jet-filter-load',
				[ $scope, JetSmartFilters, provider, query, queryID ]
			);

			if ( JetSmartFilterSettings.props && JetSmartFilterSettings.props[ provider ] && JetSmartFilterSettings.props[ provider ][ queryID ] ) {
				props = JetSmartFilterSettings.props[ provider ][ queryID ];
			}

			xhr = JetSmartFilters.ajaxRequest(
				$scope,
				'jet_smart_filters',
				provider,
				query,
				props,
				queryID
			);

		},

		ajaxRequest: function( $scope, action, provider, query, props, queryID ) {

			if ( ! queryID ) {
				queryID = 'default';
			}

			var defaults, settings, filters, controls;

			if ( JetSmartFilterSettings.queries[ provider ] ) {
				defaults = JetSmartFilterSettings.queries[ provider ][ queryID ];
			} else {
				defaults = {};
			}

			if ( JetSmartFilterSettings.settings[ provider ] ) {
				settings = JetSmartFilterSettings.settings[ provider ][ queryID ];
			} else {
				settings = {};
			}

			if ( JetSmartFilterSettings.filters[ provider ] ) {
				filters = JetSmartFilterSettings.filters[ provider ][ queryID ];
			} else {
				filters = {};
			}

			controls                 = JetSmartFilters.controls;
			JetSmartFilters.controls = false;

			$.ajax({
				url: JetSmartFilterSettings.ajaxurl,
				type: 'POST',
				dataType: 'json',
				data: {
					action: action,
					provider: provider + '/' + queryID,
					query: query,
					defaults: defaults,
					settings: settings,
					filters: filters,
					paged: JetSmartFilters.page,
					props: props,
					controls: controls,
				},
			}).done( function( response ) {

				if ( 'jet_smart_filters' === action ) {

					JetSmartFilters.renderResult( response, provider, queryID );

					$( document ).trigger(
						'jet-filter-loaded',
						[ $scope, JetSmartFilters, provider, query, queryID ]
					);

				} else {
					JetSmartFilters.renderActiveFilters( response.activeFilters, provider, queryID );
					JetSmartFilters.renderPagination( response.pagination, provider, queryID );
				}

				JetSmartFilters.page = false;

			});

		},

		renderResult: function( result, provider, queryID ) {

			if ( ! queryID ) {
				queryID = 'default';
			}

			var providerWrap = JetSmartFilterSettings.selectors[ provider ],
				$scope       = null;

			if ( 'default' === queryID ) {
				$scope = $( providerWrap.selector );
			} else {
				$scope = $( JetSmartFilters.providerSelector( providerWrap, queryID ) );
			}

			if ( 'insert' === providerWrap.action ) {
				$scope.html( result.content );
			} else {
				$scope.replaceWith( result.content );
			}

			JetSmartFilters.triggerElementorWidgets( $scope );

			$( document ).trigger(
				'jet-filter-content-rendered',
				[ $scope, JetSmartFilters, provider, queryID ]
			);

			JetSmartFilters.renderActiveFilters( result.activeFilters, provider, queryID );
			JetSmartFilters.renderPagination( result.pagination, provider, queryID );

		},

		triggerElementorWidgets : function( $scope ){

			$scope.find( 'div[data-element_type]' ).each( function() {
				var $this       = $( this ),
					elementType = $this.data( 'element_type' );

				if( 'widget' === elementType ){
					elementType = $this.data( 'widget_type' );
					window.elementorFrontend.hooks.doAction( 'frontend/element_ready/widget', $this, $ );
				}

				window.elementorFrontend.hooks.doAction( 'frontend/element_ready/' + elementType, $this, $ );

			});

		},

		renderActiveFilters: function( html, provider, queryID ) {

			if ( ! queryID ) {
				queryID = 'default';
			}

			var $activeFiltersWrap = $( 'div.jet-active-filters[data-apply-provider="' + provider + '"][data-query-id="' + queryID + '"]' );

			if ( $activeFiltersWrap.length ) {
				$activeFiltersWrap.html( html );
				$activeFiltersWrap.find( '.jet-active-filters__title' ).html(
					$activeFiltersWrap.data( 'filters-label' )
				);
			}

		},

		renderPagination: function( html, provider, queryID ) {

			if ( ! queryID ) {
				queryID = 'default';
			}

			var $paginationWrap = $( 'div.jet-smart-filters-pagination[data-apply-provider="' + provider + '"][data-query-id="' + queryID + '"]' );

			if ( $paginationWrap.length ) {
				$paginationWrap.html( html );
			}

		},

		applyFilters: function() {
			var $this    = $( this ),
				provider = $this.data( 'apply-provider' ),
				queryID  = $this.data( 'query-id' );

			JetSmartFilters.requestFilter( provider, queryID );

		},

		requestFilter: function( provider, queryID ) {

			var query = JetSmartFilters.getQuery( 'url', provider, queryID );

			if ( JetSmartFilters.page ) {
				query = JetSmartFilters.addQueryArg( 'jet_paged', JetSmartFilters.page, query, 'append' );
			}

			document.location.search = query;

		},

		getQuery: function( type, provider, queryID ) {

			var query = null;

			if ( ! queryID ) {
				queryID = 'default';
			}

			JetSmartFilters.currentQuery = null;

			if ( 'url' === type ) {
				JetSmartFilters.currentQuery = JetSmartFilters.addQueryArg(
					'jet-smart-filters',
					provider + '/' + queryID,
					null,
					'append'
				);
			} else {
				JetSmartFilters.currentQuery = {};
			}

			$( 'div[data-content-provider="' + provider + '"][data-query-id="' + queryID + '"]' ).each( function() {

				var $this          = $( this ),
					queryType      = $this.data( 'query-type' ),
					queryVar       = $this.data( 'query-var' ),
					filterType     = $this.data( 'smart-filter' ),
					key            = '_' + queryType + '_' + queryVar;

				$( document ).trigger(
					'jet-filter-add-' + filterType + '-vars',
					[ $this, key, JetSmartFilters, type ]
				);

			} );

			if ( window.elementorProFrontend && window.elementorFrontend ) {

				$.each( window.elementorFrontend.documentsManager.documents, function( index, elementorDocument ) {
					if ( 'popup' === elementorDocument.$element.data( 'elementor-type' ) ) {
						elementorDocument.$element.find( 'div[data-content-provider="' + provider + '"][data-query-id="' + queryID + '"]' ).each( function() {
							var $this          = $( this ),
								queryType      = $this.data( 'query-type' ),
								queryVar       = $this.data( 'query-var' ),
								filterType     = $this.data( 'smart-filter' ),
								key            = '_' + queryType + '_' + queryVar;

							$( document ).trigger(
								'jet-filter-add-' + filterType + '-vars',
								[ $this, key, JetSmartFilters, type ]
							);
						} );
					}
				});

			}

			query = JetSmartFilters.currentQuery;

			return query;
		},

		addQueryArg: function( key, value, query, action ) {

			key   = encodeURI( key );
			value = encodeURI( value );

			if ( ! query ) {
				query = '';
			}

			var kvp = query.split( '&' );

			if ( 'append' === action ) {

				kvp[ kvp.length ] = [ key, value ].join( '=' );

			} else {

				var i = kvp.length;
				var x;

				while ( i-- ) {
					x = kvp[ i ].split( '=' );

					if ( x[0] == key ) {
						x[1]     = value;
						kvp[ i ] = x.join( '=' );

						break;
					}
				}

				if ( i < 0 ) {
					kvp[ kvp.length ] = [ key, value ].join( '=' );
				}

			}

			return kvp.join( '&' );
		}
	};

	var JSFEProCompat = {

		archivePostsClass: '.elementor-widget-archive-posts',
		defaultPostsClass: '.elementor-widget-posts',
		postsSettings: {},
		skin: 'archive_classic',

		init: function() {
			$( document ).on( 'jet-filter-content-rendered', function( event, $scope, JetSmartFilters, provider ) {

				if ( 'epro-archive' === provider || 'epro-posts' === provider ) {

					var postsSelector = JSFEProCompat.defaultPostsClass,
						$archive = null,
						widgetName = 'posts',
						hasMasonry = false;

					if ( 'epro-archive' === provider ) {
						postsSelector = JSFEProCompat.archivePostsClass;
						widgetName = 'archive-posts';
					}

					$archive = $( postsSelector );

					JSFEProCompat.fitImages( $archive );

					JSFEProCompat.postsSettings = $archive.data( 'settings' );
					JSFEProCompat.skin          = $archive.data( 'element_type' );
					JSFEProCompat.skin          = JSFEProCompat.skin.split( widgetName + '.' );
					JSFEProCompat.skin          = JSFEProCompat.skin[1];

					hasMasonry = JSFEProCompat.postsSettings[ JSFEProCompat.skin + '_masonry' ];

					if ( 'yes' === hasMasonry ) {
						setTimeout( JSFEProCompat.initMasonry( $archive ) );
					}

				}

			} );
		},

		initMasonry: function( $archive ) {

			var $container = $archive.find( '.elementor-posts-container' ),
				$posts     = $container.find( '.elementor-post' ),
				settings   = JSFEProCompat.postsSettings,
				colsCount  = 1,
				hasMasonry = true;

			$posts.css({
				marginTop: '',
				transitionDuration: ''
			});

			var currentDeviceMode = window.elementorFrontend.getCurrentDeviceMode();

			switch ( currentDeviceMode ) {
				case 'mobile':
					colsCount = settings[ JSFEProCompat.skin + '_columns_mobile' ];
					break;
				case 'tablet':
					colsCount = settings[ JSFEProCompat.skin + '_columns_tablet' ];
					break;
				default:
					colsCount = settings[ JSFEProCompat.skin + '_columns' ];
			}

			hasMasonry = colsCount >= 2;

			$container.toggleClass( 'elementor-posts-masonry', hasMasonry );

			if ( ! hasMasonry ) {
				$container.height('');
				return;
			}

			var verticalSpaceBetween = settings[ JSFEProCompat.skin + '_row_gap' ]['size'];

			if ( ! verticalSpaceBetween ) {
				verticalSpaceBetween = settings[ JSFEProCompat.skin + '_item_gap' ]['size'];
			}

			var masonry = new elementorModules.utils.Masonry({
				container: $container,
				items: $posts.filter( ':visible' ),
				columnsCount: colsCount,
				verticalSpaceBetween: verticalSpaceBetween
			});

			masonry.run();
		},

		fitImage: function( $post ) {
			var $imageParent = $post.find( '.elementor-post__thumbnail' ),
				$image       = $imageParent.find( 'img' ),
				image        = $image[0];

			if ( ! image ) {
				return;
			}

			var imageParentRatio = $imageParent.outerHeight() / $imageParent.outerWidth(),
				imageRatio       = image.naturalHeight / image.naturalWidth;

			$imageParent.toggleClass( 'elementor-fit-height', imageRatio < imageParentRatio );
		},

		fitImages: function( $scope ) {
			var $element  = $scope,
				itemRatio = getComputedStyle( $element[0], ':after' ).content;

			$element.find( '.elementor-posts-container' ).toggleClass( 'elementor-has-item-ratio', !!itemRatio.match(/\d/) );

			$element.find( '.elementor-post' ).each( function () {
				var $post = $(this),
					$image = $post.find( '.elementor-post__thumbnail img' );

				JSFEProCompat.fitImage($post);

				$image.on( 'load', function () {
					JSFEProCompat.fitImage( $post );
				});
			} );
		},
	};

	JSFEProCompat.init();

	var JetSmartFiltersUI = {

		init: function() {

			var widgets = {
				'jet-smart-filters-range.default' : JetSmartFiltersUI.range,
				'jet-smart-filters-date-range.default' : JetSmartFiltersUI.dateRange
			};

			$.each( widgets, function( widget, callback ) {
				window.elementorFrontend.hooks.addAction( 'frontend/element_ready/' + widget, callback );
			});
		},

		range: function( $scope ) {

			var $slider = $scope.find( '.jet-range__slider' ),
				$input  = $scope.find( '.jet-range__input' ),
				$min    = $scope.find( '.jet-range__values-min' ),
				$max    = $scope.find( '.jet-range__values-max' ),
				format  = $slider.data( 'format' ),
				slider;

			if ( ! format ) {
				format = {
					'thousands_sep' : '',
					'decimal_sep' : '',
					'decimal_num' : 0,
				};
			}

			slider = $slider.slider({
				range: true,
				min: $slider.data( 'min' ),
				max: $slider.data( 'max' ),
				step: $slider.data( 'step' ),
				values: $slider.data( 'defaults' ),
				slide: function( event, ui ) {
					$input.val( ui.values[ 0 ] + ':' + ui.values[ 1 ] );

					$min.html( ui.values[ 0 ].jetFormat(
						format.decimal_num,
						3,
						format.thousands_sep,
						format.decimal_sep
					) );

					$max.html( ui.values[ 1 ].jetFormat(
						format.decimal_num,
						3,
						format.thousands_sep,
						format.decimal_sep
					) );

				},
				stop: function( event, ui ) {
					$input.trigger( 'change' );
				},
			});

		},

		dateRange: function( $scope ) {
			var $id = $scope.data('id'),
				$from  = $scope.find( '.jet-date-range__from' ),
				$to    = $scope.find( '.jet-date-range__to' ),
				$input = $scope.find( '.jet-date-range__input' ),
				from,
				to;

			from = $from.datepicker({
				defaultDate: '+1w',
				beforeShow: function (textbox, instance) {
					var $calendar = instance.dpDiv;
					$calendar.addClass('jet-smart-filters-datepicker-' + $id );
				},
				onClose: function (textbox, instance){
					var $calendar = instance.dpDiv;
					$calendar.removeClass('jet-smart-filters-datepicker-' + $id );
				}
			}).on( 'change', function() {
				to.datepicker( 'option', 'minDate', JetSmartFiltersUI.getDate( this ) );
				$input.val( $from.val() + ':' + $to.val() );
			});

			to = $to.datepicker({
				defaultDate: '+1w',
				beforeShow: function (textbox, instance) {
					var $calendar = instance.dpDiv;
					$calendar.addClass('jet-smart-filters-datepicker-' + $id );
				},
				onClose: function (textbox, instance){
					var $calendar = instance.dpDiv;
					$calendar.removeClass('jet-smart-filters-datepicker-' + $id );
				}
			}).on( 'change', function() {
				from.datepicker( 'option', 'maxDate', JetSmartFiltersUI.getDate( this ) );
				$input.val( $from.val() + ':' + $to.val() );
			});
		},

		getDate: function( element ) {

			var dateFormat = 'mm/dd/yy',
				date;

			try {
				date = $.datepicker.parseDate( dateFormat, element.value );
			} catch ( error ) {
				date = null;
			}

			return date;
		}

	};

	JetSmartFilters.init();

	$( window ).on( 'elementor/frontend/init', JetSmartFiltersUI.init );

	window.JetSmartFilters = JetSmartFilters;

}( jQuery ) );
